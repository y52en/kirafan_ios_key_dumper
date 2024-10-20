<script context="module" lang="ts">
  declare function dumpKeys(
    keychain: Uint8Array,
    manifest: Uint8Array,
    password: string,
    out: Uint8Array
  ): string;
  declare function dumpKey(
    keychain: Uint8Array,
    manifest: Uint8Array,
    password: string,
    keyType: string,
    key: string
  ): string;
</script>

<script lang="ts">
  // declareされる型だけ使いたいので、何もimportしない
  // FIXME: スマートな方法があれば修正する
  // @ts-ignore
  import type {} from "@types/golang-wasm-exec";
  import wasmUrl from "./assets/irestore.wasm?url";

  type result =
    | {
        result: "error";
        error: string;
      }
    | {
        result: "success";
        data: string;
      };

  // 他にもフィールドがあるが、今回は使わないので省略
  interface keychainData {
    v_Data: string;
  }

  let logs: string[] = [];
  function appendLog(message: string, logLevel: string = "info") {
    let prefix = `${new Date().toISOString()} [${logLevel}]`;
    logs = [...logs, `${prefix} ${message}`];
  }

  function base64ToArrayBuffer(base64: string) {
    const binary_string = atob(base64);
    const len = binary_string.length;
    const bytes = new Uint8Array(len);
    for (let i = 0; i < len; i++) {
      bytes[i] = binary_string.charCodeAt(i);
    }
    return bytes;
  }

  function downloadFile(data: Uint8Array, filename: string) {
    const blob = new Blob([data], { type: "application/octet-stream" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = filename;
    a.click();
    URL.revokeObjectURL(url);
  }

  const go = new Go();
  WebAssembly.instantiateStreaming(fetch(wasmUrl), go.importObject).then(
    (result) => {
      const module = result.module;
      const instance = result.instance;
      go.run(instance);
      appendLog("WASM loaded");
    }
  );

  let keychain: File | null = null;
  let manifest: File | null = null;
  let password = "";

  function getFileFromEvent(event: Event) {
    const target = event.target as HTMLInputElement;
    if (target && target.files) {
      return target.files[0];
    }
    return null;
  }

  function getStringFromEvent(event: Event) {
    const target = event.target as HTMLInputElement;
    if (target) {
      return target.value;
    }
    return "";
  }

  function handleKeychainFileChange(event: Event) {
    keychain = getFileFromEvent(event);
  }

  function handleManifestFileChange(event: Event) {
    manifest = getFileFromEvent(event);
  }

  function handlePasswordChange(event: Event) {
    password = getStringFromEvent(event);
  }

  async function decrypt() {
    try {
      appendLog("Decrypting. Please wait...");
      const result_json = dumpKey(
        new Uint8Array(await keychain!.arrayBuffer()),
        new Uint8Array(await manifest!.arrayBuffer()),
        password,
        "General",
        "offKey"
      );
      const result: result = JSON.parse(result_json);
      if (result.result === "error") {
        throw new Error(result.error);
      }
      appendLog(`AES KEY found! data: ${result.data}`);
      const KeyData: keychainData = JSON.parse(result.data);
      appendLog(`Your KEY: ${KeyData.v_Data}`);

      const keyBinary = base64ToArrayBuffer(KeyData.v_Data);
      downloadFile(keyBinary, "key.d");
    } catch (e: any) {
      appendLog((e as Error).message, "error");
    }
  }

  $: canSubmit = keychain && manifest && password;
</script>

<main>
  <h1>key dumper</h1>

  <div style="display: grid;gap: 20px; place-items: center;">
    <label class="cursor-select">
      keychain-backup.plist<br />
      <input type="file" on:change={handleKeychainFileChange} />
    </label>
    <label class="cursor-select">
      Manifest.plist<br />
      <input type="file" on:change={handleManifestFileChange} />
    </label>
    <input
      type="password"
      placeholder="Enter backup password"
      style="min-width: 250px; text-align: center;"
      on:input={handlePasswordChange}
    />
    <button on:click={decrypt} disabled={!canSubmit} style="min-width: 300px;">
      Decrypt
    </button>

    <span>log</span>

    <code
      style="max-width: min(100vw - 100px, 400px); width: auto; min-height: 200px; overflow-x: scroll; white-space: nowrap;"
    >
      {#each logs as log}
        <div>{log}</div>
      {/each}
    </code>
  </div>
</main>

<style>
  .cursor-select {
    cursor: pointer;
  }
</style>
