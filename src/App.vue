<template>
    <nav class="menu-bar">
        <span @click="showMenu = !showMenu">
            File
        </span>

        <span>
            Elias' Basic Text Editor
        </span>
    </nav>
    <div class="file-menu"
         v-if="showMenu">
        
        <ul>
            <li @click="onNewFile">
                New
            </li>
            <li @click="onOpenFile">
                Open
            </li>
            <li @click="onSave">
                Save As...
            </li>
            <li>
                <hr>
            </li>
            <li @click="wordWrap = !wordWrap">
                <span v-if="wordWrap">
                    ■
                </span>
                Word Wrap
            </li>
        </ul>

    </div>
    <div class="menu-overlay"
         v-if="showMenu"
         @click="showMenu = false">
    </div>

    <textarea v-model="text"
              :wrap="`${wordWrap ? '' : 'off'}`"
              :class="`${wordWrap ? '' : 'do-wrap'}`">

    </textarea>
    <nav class="status-bar">
        {{ statusBar }}
    </nav>
</template>
<script>
export default {
    data() {
        return {
            text: 'This is some sample text',
            statusBar: 'Ready',
            showMenu: false,
            wordWrap: true,
        }
    },

    methods: {
        onNewFile() {
            if(this.text != '') {
                let result = confirm("Are you sure you want to start a new file?");
                if(!result) {
                    return;
                }
            }
            this.statusBar = "New file created.";
            this.text = '';
            this.showMenu = false;
        },

        async onOpenFile() {
            const options = {
                types: [
                    {
                        description: 'Text Files',
                        accept: {
                        'text/plain': ['.txt'],
                        },
                    },
                ],
            };

            try {
                let [handle] = await window.showOpenFilePicker(options);
                let file = await handle.getFile();
                this.text = await file.text();

                this.statusBar = "Loaded file.";
            } catch(ex) {
                console.error(ex);
                this.statusBar = "Could not load file.";
            }

            this.showMenu = false;
        },

        async onSave() {
            const options = {
                types: [
                {
                    description: 'Text Files',
                    accept: {
                    'text/plain': ['.txt'],
                    },
                },
                ],
            };

            try {
                let handle = await window.showSaveFilePicker(options);

                let stream = await handle.createWritable();
                await stream.write(this.text);

                await stream.close();
                this.statusBar = `Saved at ${new Date()}`

            } catch(ex) {
                console.error(ex);
                this.statusBar = "Could not save file.";
            }

            this.showMenu = false;
        },

        onKeyDownHandler(e) {
            if(e.ctrlKey && e.key == 's') {
                e.preventDefault();
                this.onSave();
            }
            else if (e.ctrlKey && e.key == 'o') {
                e.preventDefault();
                this.onOpenFile();
            }
            else if (e.ctrlKey && e.key == 'n') {
                e.preventDefault();
                this.onOpenFile();
            }
        },
    },

    mounted() {
        document.addEventListener('keydown', this.onKeyDownHandler);
    },

    beforeUnmount() {
        document.removeEventListener('keydown', this.onKeyDownHandler);
    }
}
</script>
