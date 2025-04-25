<script>
  import { onMount } from 'svelte'
  import { obs, sendCommand } from './obs.js'
  import SourceButton from './SourceButton.svelte'

  export let programScene = {}
  export let previewScene = {}
  export let scenes = []
  export let buttonStyle = 'text' // text, screenshot, icon
  export let editable = false

  let scenesCategory = 'general'
  let scenesFiltered = []
  let scenesGroupes = {}
  let generalScenesObj = {
    playerEastHand: {},
    playerSouthHand: {},
    playerWestHand: {},
    playerNorthHand: {},
    playerEastFacial: {},
    playerNorthFacial: {},
    playerSouthFacial: {},
    playerWestFacial: {},
    playerEastReveal: {},
    playerSouthReveal: {},
    playerWestReveal: {},
    playerNorthReveal: {},
    anchors: {},
    topView: {},
    ppt: {},
    tournamentVideo: {},
    realtimeSummary: {},
    starting: {},
    resting: {},
    ending: {},
    playerEastFacialWithBgm: {},
    playerNorthFacialWithBgm: {},
    playerSouthFacialWithBgm: {},
    playerWestFacialWithBgm: {},
  }
  let lastPlayer = 0
  let isStudioMode = false
  const sceneIcons = JSON.parse(
    window.localStorage.getItem('sceneIcons') || '{}',
  )

  $: scenesFiltered = (scenesGroupes[scenesCategory] ?? []).reverse()
  // store sceneIcons on change
  $: window.localStorage.setItem('sceneIcons', JSON.stringify(sceneIcons))

  onMount(async function () {
    let data = await sendCommand('GetSceneList')
    console.log('GetSceneList', data)
    programScene = data.currentProgramSceneName || ''
    previewScene = data.currentPreviewSceneName
    scenes = data.scenes

    scenesGroupes = scenes.reduce((prev, curr) => {
      if (
        curr.sceneName.startsWith('『NINJA』') ||
        curr.sceneName.startsWith('『後台』')
      ) {
        prev['hidden'] = [...(prev['hidden'] ?? []), curr]
      } else if (curr.sceneName.startsWith('『PPT』')) {
        prev['ppt'] = [...(prev['ppt'] ?? []), curr]
      } else if (curr.sceneName.startsWith('『VIDEO』')) {
        prev['video'] = [...(prev['video'] ?? []), curr]
      } else {
        prev['general'] = [...(prev['general'] ?? []), curr]
      }

      return prev
    }, {})

    generalScenesObj = scenes.reduce(
      (prev, curr) => {
        switch (curr.sceneName) {
          case '🟩🎥 東位手牌':
            prev.playerEastHand = curr
            break
          case '🟩🎥 南位手牌':
            prev.playerSouthHand = curr
            break
          case '🟩🎥 西位手牌':
            prev.playerWestHand = curr
            break
          case '🟩🎥 北位手牌':
            prev.playerNorthHand = curr
            break
          case '🟩🎥 東北訪問':
            prev.playerEastFacial = curr
            prev.playerNorthFacial = curr
            break
          case '🟩🎥 南西訪問':
            prev.playerSouthFacial = curr
            prev.playerWestFacial = curr
            break
          // case '東家副露出街':
          //   prev.playerEastReveal = curr
          //   break
          // case '南家副露出街':
          //   prev.playerSouthReveal = curr
          //   break
          // case '西家副露出街':
          //   prev.playerWestReveal = curr
          //   break
          // case '北家副露出街':
          //   prev.playerNorthReveal = curr
          //   break
          case '🟩🎥 主播出街':
            prev.anchors = curr
            break
          case '🟩🎥 天位俯視':
            prev.topView = curr
            break
          case '🟩🌐 賽前介紹':
            prev.ppt = curr
            break
          case '🟩📼 即將開始':
            prev.starting = curr
            break
          case '🟩📼 稍後繼續':
            prev.resting = curr
            break
          case '🟩📼 規則影片':
            prev.tournamentVideo = curr
            break
          case '🟩🌐 賽事中場':
            prev.realtimeSummary = curr
            break
          case '🟩🌐 賽事結束':
            prev.ending = curr
            break
          case '🟩🎥 東北訪問':
            prev.playerEastFacialWithBgm = curr
            prev.playerNorthFacialWithBgm = curr
            break
          case '🟩🎥 南西訪問':
            prev.playerSouthFacialWithBgm = curr
            prev.playerWestFacialWithBgm = curr
            break
        }

        return prev
      },
      { ...generalScenesObj },
    )

    // data = await sendCommand('GetStudioModeEnabled')
    // if (data && data.studioModeEnabled) {
    //   isStudioMode = true
    //   previewScene = data.currentPreviewSceneName || ''
    // }
  })

  // obs.on('StudioModeStateChanged', async (data) => {
  //   console.log('StudioModeStateChanged', data.studioModeEnabled)
  //   isStudioMode = data.studioModeEnabled
  //   previewScene = programScene
  // })

  obs.on('SceneListChanged', async (data) => {
    console.log('SceneListChanged', data.scenes.length)
    scenes = data.scenes
  })

  obs.on('SceneCreated', async (data) => {
    console.log('SceneCreated', data)
  })

  obs.on('SceneRemoved', async (data) => {
    console.log('SceneRemoved', data)
    for (let i = 0; i < scenes.length; i++) {
      if (scenes[i].sceneName === data.sceneName) {
        delete scenes[i]
      }
    }
  })

  obs.on('SceneNameChanged', async (data) => {
    console.log('SceneNameChanged', data)
    for (let i = 0; i < scenes.length; i++) {
      if (scenes[i].sceneName === data.oldSceneName) {
        scenes[i].sceneName = data.sceneName
      }
    }
    // Rename in sceneIcons
    sceneIcons[data.sceneName] = sceneIcons[data.oldSceneName]
  })

  obs.on('CurrentProgramSceneChanged', (data) => {
    console.log('CurrentProgramSceneChanged', data)
    programScene = data.sceneName || ''
  })

  // obs.on('CurrentPreviewSceneChanged', async (data) => {
  //   console.log('CurrentPreviewSceneChanged', data)
  //   previewScene = data.sceneName
  // })

  function sceneClicker(scene) {
    return async function () {
      // if (isStudioMode) {
      //   await sendCommand('SetCurrentPreviewScene', {
      //     sceneName: scene.sceneName,
      //   })
      // } else {
      await sendCommand('SetCurrentProgramScene', {
        sceneName: scene.sceneName,
      })

      if (scene.sceneName === generalScenesObj.playerEastHand.sceneName) {
        lastPlayer = 0
      } else if (
        scene.sceneName === generalScenesObj.playerSouthHand.sceneName
      ) {
        lastPlayer = 1
      } else if (
        scene.sceneName === generalScenesObj.playerWestHand.sceneName
      ) {
        lastPlayer = 2
      } else if (
        scene.sceneName === generalScenesObj.playerNorthHand.sceneName
      ) {
        lastPlayer = 3
      }
      // }
    }
  }

  function nextPlayerClicker() {
    if (lastPlayer === 0) {
      sceneClicker(generalScenesObj.playerSouthHand)()
    } else if (lastPlayer === 1) {
      sceneClicker(generalScenesObj.playerWestHand)()
    } else if (lastPlayer === 2) {
      sceneClicker(generalScenesObj.playerNorthHand)()
    } else if (lastPlayer === 3) {
      sceneClicker(generalScenesObj.playerEastHand)()
    }
  }

  function onNameChange(event) {
    sendCommand('SetSceneName', {
      sceneName: event.target.title,
      newSceneName: event.target.value,
    })
  }
  function onIconChange(event) {
    sceneIcons[event.target.title] = event.target.value
  }
</script>

{#if scenesCategory === 'general' && !!generalScenesObj.playerEastHand}
  <div style="margin-bottom: 16px;text-decoration:underline;">
    <a
      href="https://hkmjbs-streaming.web.app/v1/obs/1/scene-control"
      target="_blank">多合一PPT控制台</a
    >
  </div>

  <div class="scenes-group">
    <p>純音樂、無人聲</p>
    <div class="flex">
      <div class="flex-1">
        <SourceButton
          name="開場"
          on:click={sceneClicker(generalScenesObj.starting)}
          isProgram={programScene === generalScenesObj.starting.sceneName}
          isPreview={previewScene === generalScenesObj.starting.sceneName}
          {buttonStyle}
        />
      </div>
      <div class="flex-1">
        <SourceButton
          name="過場"
          on:click={sceneClicker(generalScenesObj.resting)}
          isProgram={programScene === generalScenesObj.resting.sceneName}
          isPreview={previewScene === generalScenesObj.resting.sceneName}
          {buttonStyle}
        />
      </div>
      <div class="flex-1">
        <SourceButton
          name="完場"
          on:click={sceneClicker(generalScenesObj.ending)}
          isProgram={programScene === generalScenesObj.ending.sceneName}
          isPreview={previewScene === generalScenesObj.ending.sceneName}
          {buttonStyle}
        />
      </div>
    </div>

    <div class="flex">
      <div class="flex-1">
        <SourceButton
          name="現時數據"
          on:click={sceneClicker(generalScenesObj.realtimeSummary)}
          isProgram={programScene ===
            generalScenesObj.realtimeSummary.sceneName}
          isPreview={previewScene ===
            generalScenesObj.realtimeSummary.sceneName}
          {buttonStyle}
        />
      </div>
      <div class="flex-1"></div>
    </div>
  </div>

  <div class="flex" style="margin-top: 16px;">
    <div class="flex-1">
      <SourceButton
        name="訪問東北"
        on:click={sceneClicker(generalScenesObj.playerEastFacialWithBgm)}
        isProgram={programScene ===
          generalScenesObj.playerEastFacialWithBgm.sceneName}
        isPreview={previewScene ===
          generalScenesObj.playerEastFacialWithBgm.sceneName}
        {buttonStyle}
      />
    </div>
    <div class="flex-1">
      <SourceButton
        name="訪問西南"
        on:click={sceneClicker(generalScenesObj.playerWestFacialWithBgm)}
        isProgram={programScene ===
          generalScenesObj.playerWestFacialWithBgm.sceneName}
        isPreview={previewScene ===
          generalScenesObj.playerWestFacialWithBgm.sceneName}
        {buttonStyle}
      />
    </div>
  </div>

  <table id="general-table">
    <tr>
      <td style="width:60%"></td>
      <td colspan={2}>
        <SourceButton
          name="規則VIDEO"
          on:click={sceneClicker(generalScenesObj.tournamentVideo)}
          isProgram={programScene ===
            generalScenesObj.tournamentVideo.sceneName}
          isPreview={previewScene ===
            generalScenesObj.tournamentVideo.sceneName}
          {buttonStyle}
        /></td
      >
    </tr>
    <tr>
      <td>
        <SourceButton
          name="主播區"
          on:click={sceneClicker(generalScenesObj.anchors)}
          isProgram={programScene === generalScenesObj.anchors.sceneName}
          isPreview={previewScene === generalScenesObj.anchors.sceneName}
          {buttonStyle}
          icon={sceneIcons[generalScenesObj.anchors.sceneName] ||
            `#${Math.floor(Math.random() * 16777215).toString(16)}`}
        /></td
      >
      <td colspan="2">
        <SourceButton
          name="PPT"
          on:click={sceneClicker(generalScenesObj.ppt)}
          isProgram={programScene === generalScenesObj.ppt.sceneName}
          isPreview={previewScene === generalScenesObj.ppt.sceneName}
          {buttonStyle}
        /></td
      >
    </tr>
    <tr>
      <td colspan="3"></td>
    </tr>
    <tr>
      <td colspan={2} style="width:60%">
        <SourceButton
          name="東"
          on:click={sceneClicker(generalScenesObj.playerEastHand)}
          isProgram={programScene === generalScenesObj.playerEastHand.sceneName}
          isPreview={previewScene === generalScenesObj.playerEastHand.sceneName}
          {buttonStyle}
          icon={sceneIcons[generalScenesObj.playerEastHand.sceneName] ||
            `#${Math.floor(Math.random() * 16777215).toString(16)}`}
        /></td
      >
      <td>
        <SourceButton
          name="人"
          on:click={sceneClicker(generalScenesObj.playerEastFacial)}
          isProgram={programScene ===
            generalScenesObj.playerEastFacial.sceneName}
          isPreview={previewScene ===
            generalScenesObj.playerEastFacial.sceneName}
          {buttonStyle}
          icon={sceneIcons[generalScenesObj.playerEastFacial.sceneName] ||
            `#${Math.floor(Math.random() * 16777215).toString(16)}`}
        /></td
      >
    </tr>
    <tr>
      <td colspan={2}>
        <SourceButton
          name="南"
          on:click={sceneClicker(generalScenesObj.playerSouthHand)}
          isProgram={programScene ===
            generalScenesObj.playerSouthHand.sceneName}
          isPreview={previewScene ===
            generalScenesObj.playerSouthHand.sceneName}
          {buttonStyle}
          icon={sceneIcons[generalScenesObj.playerSouthHand.sceneName] ||
            `#${Math.floor(Math.random() * 16777215).toString(16)}`}
        /></td
      >
      <td>
        <SourceButton
          name="人"
          on:click={sceneClicker(generalScenesObj.playerSouthFacial)}
          isProgram={programScene ===
            generalScenesObj.playerSouthFacial.sceneName}
          isPreview={previewScene ===
            generalScenesObj.playerSouthFacial.sceneName}
          {buttonStyle}
          icon={sceneIcons[generalScenesObj.playerSouthFacial.sceneName] ||
            `#${Math.floor(Math.random() * 16777215).toString(16)}`}
        /></td
      >
    </tr>
    <tr>
      <td colspan={2}>
        <SourceButton
          name="西"
          on:click={sceneClicker(generalScenesObj.playerWestHand)}
          isProgram={programScene === generalScenesObj.playerWestHand.sceneName}
          isPreview={previewScene === generalScenesObj.playerWestHand.sceneName}
          {buttonStyle}
          icon={sceneIcons[generalScenesObj.playerWestHand.sceneName] ||
            `#${Math.floor(Math.random() * 16777215).toString(16)}`}
        /></td
      >
      <td>
        <SourceButton
          name="人"
          on:click={sceneClicker(generalScenesObj.playerWestFacial)}
          isProgram={programScene ===
            generalScenesObj.playerWestFacial.sceneName}
          isPreview={previewScene ===
            generalScenesObj.playerWestFacial.sceneName}
          {buttonStyle}
          icon={sceneIcons[generalScenesObj.playerWestFacial.sceneName] ||
            `#${Math.floor(Math.random() * 16777215).toString(16)}`}
        /></td
      >
    </tr>
    <tr>
      <td colspan={2}>
        <SourceButton
          name="北"
          on:click={sceneClicker(generalScenesObj.playerNorthHand)}
          isProgram={programScene ===
            generalScenesObj.playerNorthHand.sceneName}
          isPreview={previewScene ===
            generalScenesObj.playerNorthHand.sceneName}
          {buttonStyle}
          icon={sceneIcons[generalScenesObj.playerNorthHand.sceneName] ||
            `#${Math.floor(Math.random() * 16777215).toString(16)}`}
        /></td
      >
      <td>
        <SourceButton
          name="人"
          on:click={sceneClicker(generalScenesObj.playerNorthFacial)}
          isProgram={programScene ===
            generalScenesObj.playerNorthFacial.sceneName}
          isPreview={previewScene ===
            generalScenesObj.playerNorthFacial.sceneName}
          {buttonStyle}
          icon={sceneIcons[generalScenesObj.playerNorthFacial.sceneName] ||
            `#${Math.floor(Math.random() * 16777215).toString(16)}`}
        /></td
      >
    </tr>
    <tr>
      <td colspan="3">
        <SourceButton
          name="天CAM"
          on:click={sceneClicker(generalScenesObj.topView)}
          isProgram={programScene === generalScenesObj.topView.sceneName}
          isPreview={previewScene === generalScenesObj.topView.sceneName}
          {buttonStyle}
          icon={sceneIcons[generalScenesObj.topView.sceneName] ||
            `#${Math.floor(Math.random() * 16777215).toString(16)}`}
        /></td
      >
    </tr>
    <tr>
      <td colspan="3"></td>
    </tr>
    <tr>
      <td colspan="3">
        <SourceButton
          large={true}
          name="前進下一玩家"
          on:click={nextPlayerClicker}
          isProgram={false}
          isPreview={false}
          {buttonStyle}
        />
      </td>
    </tr>
  </table>
{:else}
  <ol class:column={editable} class:with-icon={buttonStyle === 'icon'}>
    {#if editable}
      {#each scenes.reverse() as scene}
        <li>
          <!-- svelte-ignore a11y-label-has-associated-control -->
          <label class="label">Name</label>
          <input
            type="text"
            class="input"
            title={scene.sceneName}
            value={scene.sceneName}
            on:change={onNameChange}
          />
          <!-- svelte-ignore a11y-label-has-associated-control -->
          <label class="label">Icon</label>
          <input
            type="text"
            class="input"
            title={scene.sceneName}
            value={sceneIcons[scene.sceneName] || ''}
            on:change={onIconChange}
          />
        </li>
      {/each}

      {#each scenesFiltered as scene}
        <li>
          <SourceButton
            name={scene.sceneName}
            on:click={sceneClicker(scene)}
            isProgram={programScene === scene.sceneName}
            isPreview={previewScene === scene.sceneName}
            {buttonStyle}
            icon={sceneIcons[scene.sceneName] ||
              `#${Math.floor(Math.random() * 16777215).toString(16)}`}
          />
        </li>
      {/each}
    {/if}
  </ol>
{/if}

<style>
  ol {
    list-style: None;
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    gap: 0.5rem;
    margin-bottom: 2rem;
  }
  ol.column {
    flex-direction: column;
    gap: 1rem;
  }
  li {
    display: inline-block;
    min-width: 10rem;
    flex-grow: 1;
  }
  ol.with-icon {
    justify-content: center;
  }
  ol.with-icon li {
    min-width: 0;
    flex-grow: 0;
    flex-shrink: 1;
  }

  #general-table {
    width: calc(100% + 8px);
    margin-left: -4px;
    margin-right: -4px;
  }

  #general-table td {
    padding: 4px;
  }

  .flex {
    display: flex;
    gap: 8px;
    margin-bottom: 8px;
  }

  .flex-1 {
    flex: 1;
  }
  .flex-2 {
    flex: 2;
  }

  .scenes-group {
    padding: 8px;
    border: 1px solid #000;
    border-radius: 4px;
  }
</style>
