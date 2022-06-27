<script setup lang="ts">
import { ref } from 'vue';
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://vuejs.org/api/sfc-script-setup.html#script-setup
import HelloWorld from './components/HelloWorld.vue'
import { open } from '@tauri-apps/api/dialog';
import { message } from '@tauri-apps/api/dialog';
import { appDir } from '@tauri-apps/api/path';
import { readDir, BaseDirectory, readBinaryFile } from '@tauri-apps/api/fs';
import { Command } from '@tauri-apps/api/shell'


// const comm = new Command('exiftool', ['-ver'])

// const comm2 = new Command('npm_version', '-v')

// comm2.on('close', data => {
//   console.log(`command finished with code ${data.code} and signal ${data.signal}`)
// });
// comm2.on('error', error => console.log(`command error: "${error}"`))
// // console.log(comm, comm2, "comm")
async function tryCommand2() {


  const command = Command.sidecar('bin/exiftool', ['-ver'])
  const output = await command.execute()

  // const command2 = Command.sidecar('binaries/exiftool', ['-ver'])
  // const command = new Command('binaries/exiftool', ['-ver']) // 
  console.log('output', output);
  command.on('close', data => console.log(data))
  command.on('error', error => console.log(`command error: "${error}"`))
  command.stdout.on('data', line => console.log(`command stdout: "${line}"`))
  command.stderr.on('data', line => console.log(`command stderr: "${line}"`))

  command.spawn()
    .then(c => {
      child = c
      console.log(child);

    })
    .catch(e => console.log(e)
    )
}
let child = null;
// function onMessage(value:string) {
//   responses.update(r => [{ text: `[${new Date().toLocaleTimeString()}]` + ': ' + (typeof value === "string" ? value : JSON.stringify(value)) }, ...r])
// }
function tryCommand(ttry: string) {
  const command = new Command('exiftool-version', [ttry]) // 
  console.log(command);
  command.on('close', data => console.log(data))
  command.on('error', error => console.log(`command error: "${error}"`))
  command.stdout.on('data', line => console.log(`command stdout: "${line}"`))
  command.stderr.on('data', line => console.log(`command stderr: "${line}"`))

  command.spawn()
    .then(c => {
      child = c
      console.log(child);

    })
    .catch(e => console.log(e)
    )
}



const objectsURL = ref<Array<string>>([])

async function selectFile() {
  const selected = await open({
    multiple: true,
    filters: [{
      name: 'Image',
      extensions: ['png', 'jpeg']
    }]
  });
  if (Array.isArray(selected)) {
    // user selected multiple files
  } else if (selected === null) {
    // user cancelled the selection
  } else {
    // user selected a single file
  }
}
async function openDirectory() {
  const selected = await open({
    directory: true,
    multiple: true,
    defaultPath: await appDir(),
  });
  if (Array.isArray(selected)) {
    console.log(selected, ' user selected multiple directories')
    // user selected multiple directories
  } else if (selected === null) {
    console.log(selected, ' user cancelled the selection')
    // user cancelled the selection
  } else {
    console.log(selected, 'user selected a single directory')
    // user selected a single directory
  }
}

async function readDirectory() {

  const selected = await open({
    directory: true,
    multiple: false,
    defaultPath: await appDir(),
  });
  if (Array.isArray(selected)) {
    console.log(selected, ' user selected multiple directories')
    // user selected multiple directories
  } else if (selected === null) {
    console.log(selected, ' user cancelled the selection')
    // user cancelled the selection
  } else {
    console.log(selected, 'user selected a single directory')
    // user selected a single directory
    const entries = await readDir(selected);
    // console.log(entries, BaseDirectory);

    entries.forEach(async (e) => {
      console.log(e.path);
      tryCommand(e.path)
      let data = await readBinaryFile(e.path);
      const blobb = new Blob([data]);
      objectsURL.value.push(URL.createObjectURL(blobb))

    })


    // const contents = await readBinaryFile(entries[0].path!);
    // console.log(contents);
    // const blobb = new Blob([contents]);
    // objectsURL.value.push(URL.createObjectURL(blobb))
    //     function processEntries(entries:[]) {
    //   for (const entry of entries) {
    //     console.log(`Entry: ${entry.path}`);
    //     if (entry.children !== null) {
    //       processEntries(entry.children)
    //     }
    //   }
    // }
  }
}



// const entries = await readDir('users', new Uint8Array([]), { dir: BaseDirectory.App, recursive: true });

// function processEntries(entries) {
//   for (const entry of entries) {
//     console.log(`Entry: ${entry.path}`);
//     if (entry.children !== null) {
//       processEntries(entry.children)
//     }
//   }
// }
// const contents = await readBinaryFile('avatar.png', { dir: BaseDirectory.Resource });
// }

</script>

<template>
  <img alt="Vue logo" src="./assets/logo.png" />
  <HelloWorld msg="Hello Vue 3 + TypeScript + Vite" /><br>
  <button @click="readDirectory">OpenDirectory</button><br>
  <button @click="tryCommand('-ver')">Command</button><br>
  <button @click="tryCommand2">Command2</button><br>
  <button @click="selectFile">selectedFile</button>
  <img v-for="src in objectsURL" style="width: 200px; height: auto; " :src="src" alt="">
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
