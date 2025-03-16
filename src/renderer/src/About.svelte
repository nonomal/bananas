<script lang="ts">
  import { L } from './translations'
  import { externalLinkClickHandler } from './Utils'
  import shoulders from './About.shoulders-of-giants.json'

  // randomize the order of the shoulders
  const randomizedShoulders = shoulders.sort(() => Math.random() - 0.5)

  let version: string
  ;(async function (): Promise<void> {
    version = await window.BananasApi.getAppVersion()
  })()

  const GITHUB_REPO_URL = 'https://github.com/mistweaverco/bananas'

  function openExternalURL(e: MouseEvent & { currentTarget: HTMLButtonElement }): void {
    const url = e.currentTarget.dataset.url
    if (!url) return

    externalLinkClickHandler(e.currentTarget, url)
  }
</script>

<div class="p-5 text-center">
  <h1 class="text-2xl">{L.about()}</h1>
  <p>You are using <code>{version}</code> of Bananas Screen Sharing</p>

  <div class="divider">Project links</div>

  <div>
    <button class="btn btn-ghost" data-url="https://getbananas.net" on:click={openExternalURL}>
      <span class="icon">
        <i class="fa-solid fa-globe"></i>
      </span>
      <strong>{L.website()}</strong>
    </button>
    <button
      class="btn btn-ghost"
      data-url="{GITHUB_REPO_URL}/issues/new"
      on:click={openExternalURL}
    >
      <span class="icon">
        <i class="fa-solid fa-bug"></i>
      </span>
      <strong>{L.report_a_bug()}</strong>
    </button>
    <button class="btn btn-ghost" data-url={GITHUB_REPO_URL} on:click={openExternalURL}>
      <span class="icon">
        <i class="fa-solid fa-code"></i>
      </span>
      <strong>{L.see_the_code()}</strong>
    </button>
    <button
      class="btn btn-ghost"
      data-url="{GITHUB_REPO_URL}/blob/main/PRIVACY.md"
      on:click={openExternalURL}
    >
      <span class="icon">
        <i class="fa-solid fa-lock"></i>
      </span>
      <strong>{L.privacty_policy()}</strong>
    </button>
    <button
      class="btn btn-ghost"
      data-url="{GITHUB_REPO_URL}/blob/main/TOS.md"
      on:click={openExternalURL}
    >
      <span class="icon">
        <i class="fa-solid fa-book"></i>
      </span>
      <strong>{L.terms_of_service()}</strong>
    </button>
    <button
      class="btn btn-ghost"
      data-url="{GITHUB_REPO_URL}/blob/main/CODE_OF_CONDUCT.md"
      on:click={openExternalURL}
    >
      <span class="icon">
        <i class="fa-solid fa-heart"></i>
      </span>
      <strong>{L.code_of_conduct()}</strong>
    </button>
  </div>

  <div class="divider">{L.shoulders_of_giants()}</div>

  <p class="pb-5">
    {L.shoulders_of_giants_description()}
  </p>

  <div class="flex flex-wrap justify-center">
    {#each randomizedShoulders as shoulder}
      <div class="card w-96 bg-base-100 card-md shadow-sm text-left">
        <div class="card-body">
          <h2 class="card-title text-accent">
            {shoulder.title}
            {shoulder.license ? '- ' + shoulder.license : ''}
          </h2>
          <p>{shoulder.description}</p>
          <p>{shoulder.usage}</p>
          <div class="justify-end card-actions">
            <a href={shoulder.url} target="_blank" rel="noopener" class="btn btn-ghost btn-sm">
              {shoulder.url}
            </a>
          </div>
        </div>
      </div>
    {/each}
  </div>
</div>
