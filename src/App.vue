<script setup lang="ts">
import { ref } from 'vue';
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://vuejs.org/api/sfc-script-setup.html#script-setup
import HelloWorld from './components/HelloWorld.vue'
import { open } from '@tauri-apps/api/dialog';
import { message } from '@tauri-apps/api/dialog';
import { appDir } from '@tauri-apps/api/path';
import { writeTextFile, readDir, BaseDirectory, readBinaryFile } from '@tauri-apps/api/fs';
import { Command } from '@tauri-apps/api/shell'


// const comm = new Command('exiftool', ['-ver'])

// const comm2 = new Command('npm_version', '-v')

// comm2.on('close', data => {
//   console.log(`command finished with code ${data.code} and signal ${data.signal}`)
// });
// comm2.on('error', error => console.log(`command error: "${error}"`))
// // console.log(comm, comm2, "comm")
async function tryCommand2(arg: string[]) {


  const command = Command.sidecar('bin/exiftool', arg)
  const output = await command.execute()

  console.log('output', output.stdout);
  // command.on('close', data => console.log(data))
  // command.on('error', error => console.log(`command error: "${error}"`))
  // command.stdout.on('data', line => console.log(`command stdout: "${line}"`))
  // command.stderr.on('data', line => console.log(`command stderr: "${line}"`))

  // command.spawn()
  //   .then(c => {
  //     child = c
  //     console.log(child);

  //   })
  //   .catch(e => console.log(e)
  //   )
  return output.stdout
}
let child = null;
// function onMessage(value:string) {
//   responses.update(r => [{ text: `[${new Date().toLocaleTimeString()}]` + ': ' + (typeof value === "string" ? value : JSON.stringify(value)) }, ...r])
// }
function tryCommand(ttry: string) {
  const command = new Command('exiftool-version', [ttry]) // 
  // console.log(command);
  command.on('close', data => console.log(data))
  command.on('error', error => console.log(`command error: "${error}"`))
  command.stdout.on('data', line => console.log(`command stdout: "${line}"`))
  command.stderr.on('data', line => console.log(`command stderr: "${line}"`))

  command.spawn()
    .then(c => {
      child = c
      console.log(child);

    })
    .catch(e => console.log(e, 'this err')
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
  console.log('click');

  const selected = await open({
    directory: true,
    multiple: false,
    // defaultPath: await appDir(),
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
    // -json "c:\Users\Phil\Images" > "c:\Users\Phil\test.json"
    const jjson = await tryCommand2(["-json", "-Rating", "-FileSize", "-FileName", selected])
    // await writeTextFile(`${selected}/a.json`, jjson);
    const entries = await readDir(selected);
    // console.log(entries, BaseDirectory);

    entries.forEach(async (e) => {
      // await writeTextFile(`${selected}/a.txt`, e.path);
      console.log(e.path);
      // tryCommand2(["-b", "-JpgFromRaw", `${e.path}`])
      tryCommand2(["-jpgfromraw", "-b", "-ext", "nef", "-w", "jpeg", `${e.path}`])
      tryCommand2(["-tagsfromfile", `${e.path}`, " -ext", "jpeg"])

      // let data = await readBinaryFile(e.path);
      // const blobb = new Blob([data]);
      // objectsURL.value.push(URL.createObjectURL(blobb))

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
  <!-- <img alt="Vue logo" src="./assets/logo.png" /> -->
  <HelloWorld msg="Photo rayting" /><br>
  <button @click="readDirectory">OpenDirectory</button><br>
  <!-- <button @click="tryCommand('-ver')">Command</button><br> -->
  <button @click="tryCommand2('-ver')">Command2</button><br>
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
