<script lang="ts">
  import { useActiveView, useNavigationEnabled, useIsWatching, useIsHosting } from './stores'
  import { L } from './translations'
  const activeView = useActiveView()
  const isHosting = useIsHosting()
  const isWatching = useIsWatching()
  const navigationEnabled = useNavigationEnabled()

  const handleTopButtonsClick = (evt: MouseEvent): void => {
    evt.preventDefault()
    const target = evt.target as HTMLButtonElement
    const root = target.closest('button')

    $activeView = root.dataset.action
  }
</script>

<div class="navbar bg-base-100 shadow-sm">
  <button
    class="btn {$activeView === 'join' ? 'btn-secondary' : 'btn-ghost'}"
    data-action="join"
    on:click={handleTopButtonsClick}
    disabled={!$navigationEnabled}
  >
    <span class="icon">
      <i class="fa-solid fa-right-to-bracket"></i>
    </span>
    <strong>{!$isWatching ? L.join_a_session() : L.joined_a_session()}</strong>
  </button>
  <button
    class="btn {$activeView === 'host' ? 'btn-secondary' : 'btn-ghost'}"
    data-action="host"
    on:click={handleTopButtonsClick}
    disabled={!$navigationEnabled}
  >
    <span class="icon">
      <i class="fa-solid fa-earth-africa"></i>
    </span>
    <strong>{!$isHosting ? L.host_a_session() : L.hosting_a_session()}</strong>
  </button>
  <button
    class="btn {$activeView === 'settings' ? 'btn-secondary' : 'btn-ghost'}"
    data-action="settings"
    on:click={handleTopButtonsClick}
    disabled={!$navigationEnabled}
  >
    <span class="icon">
      <i class="fa-solid fa-gear"></i>
    </span>
    <strong>{L.settings()}</strong>
  </button>
  <button
    class="btn {$activeView === 'about' ? 'btn-secondary' : 'btn-ghost'}"
    data-action="about"
    on:click={handleTopButtonsClick}
    disabled={!$navigationEnabled}
  >
    <span class="icon">
      <i class="fa-solid fa-question"></i>
    </span>
    <strong>{L.about()}</strong>
  </button>
</div>
