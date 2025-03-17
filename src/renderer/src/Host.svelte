<script lang="ts">
  import { onMount } from 'svelte'
  import { L } from './translations'
  import { useNavigationEnabled, useIsHosting, useHostUrl } from './stores'
  import { mayBeConnectionString, getDataFromBananasUrl, ConnectionType } from './Utils'
  import AudioVisualizer from './AudioVisualizer.svelte'
  import WebRTC from './WebRTC.svelte'

  const navigationEnabled = useNavigationEnabled()
  const isHosting = useIsHosting()

  let webRTCComponent: WebRTC
  let connectButton: HTMLButtonElement
  let copyButton: HTMLButtonElement

  let connectionState = 'disconnected'
  let cursorsActive = false
  let displayStreamActive = false
  let microphoneActive = false
  let isStreaming = false
  let sessionStarted = false
  let connectionStringIsValid: boolean | null = null
  let connectToUserName = ''
  let copyButtonIsLoading = false
  let connectionString = useHostUrl()
  let hasAudioInput = false
  let visualizerIsActive: boolean = true

  const onConnectionStringChange = async (): Promise<void> => {
    if ($connectionString === '') {
      connectionStringIsValid = null
      return
    }
    connectionStringIsValid = mayBeConnectionString(ConnectionType.PARTICIPANT, $connectionString)
    if (connectionStringIsValid) {
      const banansData = await getDataFromBananasUrl($connectionString)
      connectToUserName = banansData.data.username
    }
  }

  const onConnectionStateChange = (): void => {
    switch (connectionState) {
      case 'connected':
        // TODO: show a toast
        break
      case 'failed':
        // TODO: show a toast
        break
      case 'closed':
        // TODO: show a toast
        break
      default:
        break
    }
  }

  $: $connectionString, onConnectionStringChange()
  $: connectionState, onConnectionStateChange()

  const toggleRemoteCursors = (): void => {
    cursorsActive = !cursorsActive
    window.BananasApi.toggleRemoteCursors(cursorsActive)
    webRTCComponent.ToggleRemoteCursors(cursorsActive)
  }

  onMount(async () => {
    const settings = await window.BananasApi.getSettings()
    microphoneActive = settings.isMicrophoneEnabledOnConnect
    connectButton.addEventListener('click', async () => {
      const data = await getDataFromBananasUrl($connectionString)
      await webRTCComponent.Connect(data.rtcSessionDescription)
      isStreaming = true
      displayStreamActive = true
    })
    copyButton.addEventListener('click', async () => {
      copyButtonIsLoading = true
      const offer = await webRTCComponent.CreateHostUrl({
        username: settings.username
      })
      navigator.clipboard.writeText(offer)
      setTimeout(() => {
        copyButtonIsLoading = false
      }, 400)
    })
  })
  const onStartSessionButtonClick = async (): Promise<void> => {
    await webRTCComponent.Setup()
    sessionStarted = true
    $navigationEnabled = false
    $isHosting = true
    hasAudioInput = webRTCComponent.HasAudioInput()
  }
  const reset = (): void => {
    $connectionString = ''
    cursorsActive = false
    displayStreamActive = false
    microphoneActive = true
    isStreaming = false
    sessionStarted = false
    connectionStringIsValid = null
    copyButtonIsLoading = false
    $navigationEnabled = true
    $isHosting = false
  }
  const onDisconnectClick = async (): Promise<void> => {
    await webRTCComponent.Disconnect()
    reset()
  }
  const onMicrophoneToggle = async (): Promise<void> => {
    microphoneActive = !microphoneActive
    webRTCComponent.ToggleMicrophone()
  }
  const onDisplayStreamToggle = async (): Promise<void> => {
    displayStreamActive = !displayStreamActive
    webRTCComponent.ToggleDisplayStream()
    if (!displayStreamActive) {
      cursorsActive = false
      window.BananasApi.toggleRemoteCursors(cursorsActive)
      webRTCComponent.ToggleRemoteCursors(cursorsActive)
    }
  }
</script>

<WebRTC bind:connectionState bind:this={webRTCComponent} />

<div class="container">
  <h1 class="title">{!isStreaming ? L.host_a_session() : L.hosting_a_session()}</h1>
  <div class={!isStreaming ? 'hidden' : ''}>
    <div class="btn {displayStreamActive ? 'btn-success' : 'btn-warning'}">
      <button
        aria-label={displayStreamActive ? L.streaming_your_display() : L.streaming_your_display()}
        title={displayStreamActive ? L.streaming_your_display() : L.streaming_your_display()}
        on:click={onDisplayStreamToggle}
      >
        <span class="icon">
          <i class="fa-solid fa-display"></i>
        </span>
      </button>
      {#if hasAudioInput}
        <div class="btn {microphoneActive ? 'btn-success' : 'btn-warning'}">
          <button
            title={microphoneActive ? 'Microphone active' : 'Microphone muted'}
            on:click={onMicrophoneToggle}
          >
            <span class="icon">
              {#if microphoneActive}
                <AudioVisualizer
                  className="icon {!visualizerIsActive ? 'hidden' : ''}"
                  bind:visualizerIsActive
                  stream={webRTCComponent.GetAudioStream()}
                />
                <i class="fas fa-microphone {visualizerIsActive ? 'hidden' : ''}"></i>
              {:else}
                <i class="fas fa-microphone-slash"></i>
              {/if}
            </span>
          </button>
        </div>
      {/if}

      <div
        class="button {cursorsActive ? 'is-success' : 'is-danger'} {!displayStreamActive
          ? 'hidden'
          : ''}"
      >
        <button
          aria-label={cursorsActive ? L.remote_cursors_enabled() : L.remote_cursors_disabled()}
          title={cursorsActive ? L.remote_cursors_enabled() : L.remote_cursors_disabled()}
          on:click={toggleRemoteCursors}
        >
          <span class="icon">
            <i class="fas fa-mouse-pointer"></i>
          </span>
        </button>
      </div>
    </div>
    <div class="btn btn-warning">
      <button on:click={onDisconnectClick}>
        <span class="icon">
          <i class="fas fa-unlink"></i>
        </span>
        <span>{L.disconnect()}</span>
      </button>
    </div>
  </div>

  <div class="btn btn-info {!sessionStarted ? '' : 'hidden'}">
    <button disabled={sessionStarted} on:click={onStartSessionButtonClick}>
      <span class="icon">
        <i class="fas fa-play"></i>
      </span>
      <span>{!sessionStarted ? L.start_a_new_session() : L.session_started()}</span>
    </button>
  </div>

  <div class="btn btn-warning {!sessionStarted || isStreaming ? 'hidden' : ''}">
    <button on:click={onDisconnectClick}>
      <span class="icon">
        <i class="fas fa-unlink"></i>
      </span>
      <span>{L.cancel()}</span>
    </button>
  </div>

  <div class="btn btn-info {!sessionStarted || isStreaming ? 'hidden' : ''}">
    <button bind:this={copyButton}>
      <span class="icon">
        <i class="fas fa-copy"></i>
      </span>
      <span>{L.copy_my_connection_string()}</span>
    </button>
  </div>
</div>

<div class="container {!sessionStarted || isStreaming ? 'hidden' : ''}">
  <div
    class="input {connectionStringIsValid ? '' : 'btn-disabled'} {connectionStringIsValid === null
      ? ''
      : connectionStringIsValid
        ? 'input-success'
        : 'input-warning'}"
  >
    <input bind:value={$connectionString} placeholder="participant connection string" type="text" />
    <span class="icon is-small is-left">
      <i class="fas fa-user"></i>
    </span>
    <span class="icon is-small is-right">
      <i
        class="fas fa-question {connectionStringIsValid === null
          ? 'fa-question'
          : connectionStringIsValid
            ? 'fa-check'
            : 'fa-times'}"
      ></i>
    </span>
  </div>
  <div
    class="btn {connectionStringIsValid ? '' : 'btn-disabled'} {connectionStringIsValid === null
      ? 'btn-info'
      : connectionStringIsValid
        ? 'btn-success'
        : 'btn-warning'}"
  >
    <button bind:this={connectButton} disabled={connectionStringIsValid ? false : true}>
      <span class="icon">
        <i class="fas fa-link"></i>
      </span>
      <span>{L.connect()} {connectionStringIsValid ? connectToUserName : ''} </span>
    </button>
  </div>
</div>
