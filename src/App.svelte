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
</script>

<main>
  <div>
    <a href="https://vitejs.dev" target="_blank" rel="noreferrer">
      <img src={viteLogo} class="logo" alt="Vite Logo" />
    </a>
    <a href="https://svelte.dev" target="_blank" rel="noreferrer">
      <img src={svelteLogo} class="logo svelte" alt="Svelte Logo" />
    </a>
  </div>
  <h1>Vite + Svelte</h1>

  <div class="card">
    <Counter />
  </div>

  <p>
    Check out <a href="https://github.com/sveltejs/kit#readme" target="_blank" rel="noreferrer">SvelteKit</a>, the official Svelte app framework powered by Vite!
  </p>

  <p class="read-the-docs">
    Click on the Vite and Svelte logos to learn more
  </p>
</main>

<style>
  .logo {
    height: 6em;
    padding: 1.5em;
    will-change: filter;
    transition: filter 300ms;
  }
  .logo:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
  }
  .logo.svelte:hover {
    filter: drop-shadow(0 0 2em #ff3e00aa);
  }
  .read-the-docs {
    color: #888;
  }
</style>
