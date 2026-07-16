<template>
  <div>
    <h2 class="mb-4">MOS LSI Tools</h2>

    <div style="margin-bottom: 80px">
      <v-card class="mb-4 pa-0">
        <v-card-title class="d-flex align-center">
          <v-icon class="mr-2">mdi-information-outline</v-icon>
          <span>Overview</span>
        </v-card-title>
        <v-card-text>
          <p class="text-medium-emphasis mb-4">
            Broadcom/LSI command-line utilities for managing, flashing and monitoring Host Bus Adapters (HBAs)
            from the MOS terminal.
          </p>
          <span class="text-subtitle-1 font-weight-medium d-block mb-2">Included tools</span>
          <v-chip class="mr-2 mb-2" size="small" variant="tonal">storcli64</v-chip>
          <v-chip class="mr-2 mb-2" size="small" variant="tonal">sas3flash</v-chip>
          <v-chip class="mr-2 mb-2" size="small" variant="tonal">megacli64</v-chip>
        </v-card-text>
      </v-card>

      <v-container fluid class="pt-2 pr-0 pl-0 pb-2">
        <div class="d-flex align-center ga-3 mb-4">
          <div style="width: 4px; height: 32px; border-radius: 2px; background: rgb(var(--v-theme-primary))"></div>
          <h2 class="font-weight-medium ma-0" style="font-weight: 600; line-height: 1.1">Terminal</h2>
          <v-spacer></v-spacer>
          <v-btn
            v-if="!terminalActive"
            text class="d-flex align-center" density="compact"
            @click="openTerminal"
          >
            <v-icon small class="mr-1">mdi-console</v-icon>
            Open terminal
          </v-btn>
          <v-btn
            v-else
            text class="d-flex align-center" density="compact" color="error"
            @click="closeTerminal"
          >
            <v-icon small class="mr-1">mdi-close</v-icon>
            Close terminal
          </v-btn>
        </div>
        <p v-if="!terminalActive" class="text-medium-emphasis ml-5">
          Open a shell to run storcli64, sas3flash and megacli64 directly from this page.
        </p>
        <div v-else>
          <div ref="terminalContainer" style="width: 100%; height: 420px; padding: 8px; background: #000000; border-radius: 8px"></div>
        </div>
      </v-container>

      <v-card class="mb-4 pa-0">
        <v-card-title class="d-flex align-center">
          <v-icon class="mr-2">mdi-console</v-icon>
          <span>storcli64</span>
        </v-card-title>
        <v-card-text>
          <pre class="cmd-block"># Show summary of all controllers
storcli64 show

# Show detailed information for Controller 0
storcli64 /c0 show</pre>
        </v-card-text>
      </v-card>

      <v-card class="mb-4 pa-0">
        <v-card-title class="d-flex align-center">
          <v-icon class="mr-2">mdi-console</v-icon>
          <span>sas3flash</span>
        </v-card-title>
        <v-card-text>
          <pre class="cmd-block"># List all Broadcom/LSI adapters detected in the system
sas3flash -listall

# Show detailed information for the primary adapter
sas3flash -list</pre>
        </v-card-text>
      </v-card>

      <v-card class="mb-4 pa-0">
        <v-card-title class="d-flex align-center">
          <v-icon class="mr-2">mdi-console</v-icon>
          <span>megacli64</span>
        </v-card-title>
        <v-card-text>
          <pre class="cmd-block"># View detailed information about all RAID adapters
megacli64 -AdpAllInfo -aALL

# List all physical drives connected to the controllers
megacli64 -PDList -aALL</pre>
        </v-card-text>
      </v-card>

      <v-card class="mb-4 pa-0">
        <v-card-title class="d-flex align-center">
          <v-icon class="mr-2">mdi-book-open-variant</v-icon>
          <span>References</span>
        </v-card-title>
        <v-card-text>
          <v-list density="compact" class="pa-0">
            <v-list-item
              href="https://github.com/MSU-Libraries/storcli-doc"
              target="_blank"
              rel="noopener noreferrer"
              prepend-icon="mdi-open-in-new"
              title="Nate Colin's StorCLI Documentation"
            />
            <v-list-item
              href="https://docs.broadcom.com/doc/12352476"
              target="_blank"
              rel="noopener noreferrer"
              prepend-icon="mdi-open-in-new"
              title="Broadcom StorCLI Reference Guide"
            />
            <v-list-item
              href="https://support.hpe.com/hpesc/public/docDisplay?docId=a00048288en_us&docLocale=en_US"
              target="_blank"
              rel="noopener noreferrer"
              prepend-icon="mdi-open-in-new"
              title="HPE StorCLI Reference Guide"
            />
            <v-list-item
              href="https://www.ibm.com/docs/en/power9/0009-ESS?topic=22p-storcli-commands"
              target="_blank"
              rel="noopener noreferrer"
              prepend-icon="mdi-open-in-new"
              title="IBM Power9 StorCLI Manual"
            />
          </v-list>
        </v-card-text>
      </v-card>
    </div>
  </div>
</template>

<script setup>
import { ref, onUnmounted, nextTick } from 'vue';
import { Terminal } from '@xterm/xterm';
import { FitAddon } from '@xterm/addon-fit';
import { ClipboardAddon } from '@xterm/addon-clipboard';
import { io } from 'socket.io-client';
import '@xterm/xterm/css/xterm.css';

let resizeHandler = null;
const terminalContainer = ref(null);
const terminalActive = ref(false);
let term = null;
let fitAddon = null;
let clipboardAddon = null;
let socket = null;
let terminalSessionId = null;

const getAuthHeaders = () => ({
  Authorization: 'Bearer ' + localStorage.getItem('authToken'),
});

const openTerminal = async () => {
  terminalActive.value = true;
  await nextTick();

  try {
    const res = await fetch('/api/v1/terminal/create', {
      method: 'POST',
      headers: {
        ...getAuthHeaders(),
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        cwd: '/root',
      }),
    });

    if (!res.ok) {
      throw new Error('Failed to create terminal session');
    }

    const session = await res.json();
    terminalSessionId = session.sessionId;

    fitAddon = new FitAddon();
    clipboardAddon = new ClipboardAddon();

    term = new Terminal({ cursorBlink: true, fontFamily: 'monospace', fontSize: 14 });
    term.loadAddon(fitAddon);
    term.loadAddon(clipboardAddon);
    term.open(terminalContainer.value);
    term.focus();
    fitAddon.fit();

    term.onSelectionChange(() => {
      if (!term.hasSelection()) return;
      const selected = term.getSelection();

      if (navigator.clipboard && window.isSecureContext) {
        navigator.clipboard.writeText(selected);
      } else {
        const textarea = document.createElement('textarea');
        textarea.value = selected;
        textarea.style.cssText = 'position:fixed;opacity:0';
        document.body.appendChild(textarea);
        textarea.focus();
        textarea.select();
        try {
          document.execCommand('copy');
        } catch (e) {}
        document.body.removeChild(textarea);
      }
    });

    socket = io('/terminal', { path: '/api/v1/socket.io/' });

    socket.on('connect', () => {
      socket.emit('join-session', {
        sessionId: terminalSessionId,
        token: localStorage.getItem('authToken'),
      });
    });

    socket.on('session-joined', (info) => {
      term.resize(info.cols, info.rows);
      fitAddon.fit();
      socket.emit('terminal-resize', { cols: term.cols, rows: term.rows });
    });

    socket.on('terminal-output', (data) => {
      term.write(data);
    });

    term.onData((data) => {
      socket.emit('terminal-input', data);
    });

    term.onResize(({ cols, rows }) => {
      socket.emit('terminal-resize', { cols, rows });
    });

    resizeHandler = () => {
      if (fitAddon) fitAddon.fit();
    };
    window.addEventListener('resize', resizeHandler);

    socket.on('disconnect', () => {
      if (term) term.write('\r\nConnection closed.\r\n');
    });
  } catch (e) {
    console.error('Failed to open terminal:', e);
    terminalActive.value = false;
  }
};

const closeTerminal = () => {
  if (resizeHandler) {
    window.removeEventListener('resize', resizeHandler);
    resizeHandler = null;
  }
  if (socket) {
    socket.emit('leave-session');
    socket.disconnect();
    socket = null;
  }
  if (term) {
    term.dispose();
    term = null;
  }
  if (fitAddon) {
    fitAddon.dispose();
    fitAddon = null;
  }
  if (clipboardAddon) {
    clipboardAddon.dispose();
    clipboardAddon = null;
  }
  terminalSessionId = null;
  terminalActive.value = false;
};

onUnmounted(() => {
  closeTerminal();
});
</script>

<style scoped>
.cmd-block {
  margin: 0;
  padding: 12px;
  border-radius: 4px;
  font-family: monospace;
  font-size: 0.875rem;
  line-height: 1.5;
  white-space: pre-wrap;
  overflow-x: auto;
  background: rgba(var(--v-theme-on-surface), 0.05);
}
</style>
