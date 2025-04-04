<script lang="ts">
  import Action from "$lib/components/Action.svelte";
  import { popup } from "$lib/popup";
  import { deviceMeta, serialPort } from "$lib/serial/connection";
  import { setting } from "$lib/setting";
  import ResetPopup from "./ResetPopup.svelte";
  import LL from "$i18n/i18n-svelte";
  import {
    createChordBackup,
    createLayoutBackup,
    createSettingsBackup,
    downloadBackup,
    downloadFile,
    restoreBackup,
    restoreFromFile,
  } from "$lib/backup/backup";
  import { preference } from "$lib/preferences";
  import { action } from "$lib/title";
  import { fly } from "svelte/transition";
</script>

<svelte:head>
  <title>Device Settings - CharaChorder Device Manager</title>
  <meta name="description" content="Change your device's settings" />
</svelte:head>

<section>
  <fieldset>
    <legend>{$LL.backup.TITLE()}</legend>
    <label
      ><input
        type="checkbox"
        use:preference={"backup"}
      />{$LL.backup.AUTO_BACKUP()}</label
    >
    <p class="disclaimer">
      {$LL.backup.DISCLAIMER()}
    </p>
    <div class="row" style="margin-top: auto">
      <button onclick={() => downloadFile(createChordBackup())}>
        <span class="icon">piano</span>
        {$LL.configure.chords.TITLE()}
      </button>
      <button onclick={() => downloadFile(createLayoutBackup())}>
        <span class="icon">keyboard</span>
        {$LL.configure.layout.TITLE()}
      </button>
      <button onclick={() => downloadFile(createSettingsBackup())}>
        <span class="icon">settings</span>
        Settings
      </button>
    </div>
    <div class="row">
      <button class="primary" onclick={downloadBackup}
        ><span class="icon">download</span>{$LL.backup.DOWNLOAD()}</button
      >
      <label class="button"
        ><input oninput={restoreBackup} type="file" /><span class="icon"
          >settings_backup_restore</span
        >{$LL.backup.RESTORE()}</label
      >
    </div>
  </fieldset>

  <fieldset>
    <legend>Device</legend>
    <label
      >{$LL.deviceManager.AUTO_CONNECT()}<input
        type="checkbox"
        use:preference={"autoConnect"}
      /></label
    >
    {#if $serialPort}
      <label
        >Boot message<input type="checkbox" use:setting={{ id: 0x93 }} /></label
      >
      <label
        >GTM Realtime Feedback<input
          type="checkbox"
          use:setting={{ id: 0x92 }}
        /></label
      >
      {#if $deviceMeta?.factoryDefaults?.settings}
        <button
          use:action={{ title: "Reset Settings" }}
          transition:fly={{ x: -8 }}
          onclick={() => restoreFromFile($deviceMeta.factoryDefaults!.settings)}
          ><span class="icon">reset_settings</span>Reset Settings</button
        >
      {/if}
      <button class="outline" use:popup={ResetPopup}>Recovery...</button>
    {/if}
  </fieldset>

  {#if $serialPort}
    <fieldset>
      <legend
        ><label
          ><input type="checkbox" use:setting={{ id: 0x41 }} />Spurring</label
        ></legend
      >
      <p>
        "Chording only" mode which tells your device to output chords on a press
        rather than a press & release. It also enables you to jump from one
        chord to another without releasing everything and can be activated in
        GTM or by chording both mirror keys. It can provide significant speed
        gains with chording, but also takes away the flexibility of character
        entry.
      </p>
      <p>
        Spurring also helps new users learn how to chord by eliminating the need
        to focus on timing.
      </p>
      <p>
        Spurring is toggled by chording <Action display="keys" action={540} /> and
        <Action display="keys" action={542} /> together.
      </p>
      <label
        >Character Counter Timeout<span class="unit"
          ><input
            type="number"
            step="0.001"
            min="0"
            max="240"
            use:setting={{ id: 0x43, scale: 0.001 }}
          />s</span
        ></label
      >
    </fieldset>

    <fieldset>
      <legend
        ><label
          ><input
            type="checkbox"
            use:setting={{ id: 0x51 }}
          />Arpeggiates</label
        ></legend
      >
      <p>
        A quick, single key press and release used to indicate a suffix, prefix,
        or modifier to be associated with a chord.
      </p>
      <p>
        The following keys have special behavior when arpeggiates are enabled:
      </p>
      <ul>
        <li>
          <Action display="keys" action={44} />, <Action
            display="keys"
            action={59}
          /> and <Action display="keys" action={58} /> will be placed before the
          auto-inserted space
        </li>
        <li>
          <Action display="keys" action={46} />, <Action
            display="keys"
            action={63}
          /> and <Action display="keys" action={33} /> will be placed before the
          auto-inserted space and capitalize the next word
        </li>
        <li>
          <Action display="keys" action={45} /> and <Action
            display="keys"
            action={47}
          /> will replace the auto-inserted space
        </li>
      </ul>
      <label
        >Timeout After Chord<span class="unit"
          ><input type="number" step="1" use:setting={{ id: 0x54 }} />ms</span
        ></label
      >
    </fieldset>

    <fieldset>
      <legend>Chord Modifiers</legend>
      <p>
        Chord modifiers change a chord when held with the chord or when pressed
        after (arpeggiated), <b>provided that arpeggiates are enabled.</b>
      </p>
      <ul>
        <li>
          <Action display="keys" action={513} /> Capitalizes the first letter of
          a chord
        </li>
        <li>
          <Action display="keys" action={540} /> Present Tense (supported words only)
        </li>
        <li>
          <Action display="keys" action={542} /> Plural (supported words only)
        </li>
        <li>
          <Action display="keys" action={550} /> Past Tense (supported words only)
        </li>
        <li>
          <Action display="keys" action={551} /> Comparative (supported words only)
        </li>
      </ul>
    </fieldset>

    <fieldset>
      <legend>Character Entry</legend>
      {#if $serialPort.device === "LITE"}
        <label
          >Swap Keymap 0 and 1<input
            type="checkbox"
            use:setting={{ id: 0x13 }}
          /></label
        >
      {/if}
      <label>
        Character Entry (chentry)
        <input type="checkbox" use:setting={{ id: 0x12 }} />
      </label>
      <label>
        Key Scan Rate
        <span class="unit"
          ><input
            type="number"
            use:setting={{ id: 0x14, inverse: 1000 }}
          />Hz</span
        ></label
      >
      <label>
        Key Debounce Press<span class="unit"
          ><input type="number" use:setting={{ id: 0x15 }} />ms</span
        ></label
      >
      <label
        >Key Debounce Release<span class="unit"
          ><input type="number" use:setting={{ id: 0x16 }} />ms</span
        ></label
      >
      <label
        >Output Character Delay<span class="unit"
          ><input type="number" use:setting={{ id: 0x17 }} />µs</span
        ></label
      >
    </fieldset>

    <fieldset>
      <legend
        ><label><input type="checkbox" use:setting={{ id: 0x21 }} />Mouse</label
        ></legend
      >
      <label
        >Mouse Speed<input type="number" use:setting={{ id: 0x22 }} /><input
          type="number"
          use:setting={{ id: 0x23 }}
        /></label
      >
      <label
        >Scroll Speed<input type="number" use:setting={{ id: 0x25 }} /></label
      >
      <label>
        <span>
          Active Mouse
          <p>Bounces mouse by 1px every 60s if enabled</p></span
        >
        <input type="checkbox" use:setting={{ id: 0x24 }} /></label
      >
      <label
        >Poll Rate<span class="unit"
          ><input
            type="number"
            use:setting={{ id: 0x26, inverse: 1000 }}
          />Hz</span
        ></label
      >
    </fieldset>

    <fieldset>
      <legend
        ><label
          ><input type="checkbox" use:setting={{ id: 0x31 }} />Chording</label
        ></legend
      >
      <label
        >Auto-delete Timeout <span class="unit"
          ><input
            type="number"
            min="0"
            max="25500"
            step="10"
            use:setting={{ id: 0x33 }}
          />ms</span
        ></label
      >
      <label
        >Press Tolerance<span class="unit"
          ><input
            type="number"
            min="1"
            max="150"
            step="1"
            use:setting={{ id: 0x34 }}
          />ms</span
        ></label
      >
      <label
        >Release Tolerance<span class="unit"
          ><input
            type="number"
            min="1"
            max="150"
            step="1"
            use:setting={{ id: 0x35 }}
          />ms</span
        ></label
      >
    </fieldset>

    {#if $serialPort.device === "LITE"}
      <fieldset>
        <legend
          ><label><input type="checkbox" use:setting={{ id: 0x84 }} />RGB</label
          ></legend
        >
        <label
          >Brightness<input
            use:setting={{ id: 0x81 }}
            type="number"
            min="0"
            max="50"
            step="1"
          /></label
        >
        <select use:setting={{ id: 0x82 }}>
          <option value="0">White</option>
          <option value="1">Red</option>
          <option value="2">Orange</option>
          <option value="3">Yellow</option>
          <option value="4">Lime</option>
          <option value="5">Green</option>
          <option value="7">Cyan</option>
          <option value="9">Blue</option>
          <option value="10">Violet</option>
          <option value="11">Pink</option>
          <option value="13">Multicolor</option>
        </select>
      </fieldset>
    {/if}
  {/if}
</section>

<style lang="scss">
  section {
    overflow-y: auto;
    display: flex;
    flex-flow: row wrap;
    gap: 16px;
    justify-content: center;

    margin-block: auto;
    padding-block-end: 48px;
  }

  button.outline {
    border: 1px solid currentcolor;
    border-radius: 8px;
    height: 2em;
    margin-block: 2em;
    margin-inline: auto;
  }

  legend,
  legend > label {
    font-size: 24px;
    font-weight: bold;
    position: relative;
    padding: 0 16px;
  }

  legend:has(label) {
    padding: 0;
  }

  legend:not(:has(label)) {
    opacity: 0.8;
  }

  input[type="checkbox"] {
    font-size: 12px !important;
  }

  fieldset {
    display: flex;
    flex-direction: column;

    max-width: 400px;
    border: 1px solid var(--md-sys-color-outline);
    border-radius: 24px;

    /*&:has(> legend input:not(:checked)) > :not(legend) {
      pointer-events: none;
      opacity: 0.7;
    }*/

    > label {
      position: relative;
      display: flex;
      gap: 16px;
      align-items: center;
      justify-content: space-between;

      margin-block: 4px;

      font-size: 14px;

      > input[type="number"] {
        border-radius: 16px 4px 4px 16px;
        height: 24px;
        text-align: center;

        &:last-child:not(:only-child) {
          border-radius: 4px 16px 16px 4px;
        }

        &:only-child {
          border-radius: 16px;
        }
      }

      &:has(input[type="number"]) {
        cursor: text;

        &:hover {
          filter: none;
        }
      }
    }

    .unit {
      overflow: hidden;
      display: flex;
      gap: 4px;
      align-items: center;
      justify-content: flex-start;

      width: 67px;
      padding-inline-end: auto;

      font-size: 12px;
      font-weight: bold;

      background: var(--md-sys-color-secondary-container);
      border-radius: 16px;
    }

    input[type="number"] {
      display: flex;

      width: 5ch;
      height: 100%;
      padding-block: 4px;

      font-family: "Noto Sans Mono", monospace;
      color: var(--md-sys-color-on-secondary);
      text-align: end;

      background: var(--md-sys-color-secondary);
      border: none;

      &::-webkit-inner-spin-button {
        display: none;
      }

      &::after {
        content: "bleh";
      }

      &:focus {
        outline: none;
      }
    }

    ul,
    p {
      font-size: 10px;

      :global(kbd) {
        font-size: 12px;
        height: 18px;
      }
    }
  }

  // stylelint-disable-next-line
  label:global(:has(.pending-changes)) {
    color: var(--md-sys-color-primary);

    &::before {
      position: absolute;
      font-size: 16px;
      top: 0.5em;
      right: 0.25em;
      content: "•";
    }
  }

  .row {
    display: flex;
    justify-content: space-evenly;
    margin-block: 8px;
  }

  input[type="file"] {
    display: none;
  }
</style>
