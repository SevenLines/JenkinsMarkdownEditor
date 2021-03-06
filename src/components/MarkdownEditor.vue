<template>
    <textarea ref="editor"></textarea>
</template>

<script lang="ts">
    import {Component, Prop, Vue, Watch} from "vue-property-decorator";
    import EasyMDE from 'easymde';
    import marked from 'marked'

    @Component
    export default class MarkdownEditor extends Vue {
        @Prop() value!: string;
        @Prop() maxHeight!: "300px";
        @Prop() minHeight!: "300px";
        @Prop() uploadLocalFuncPath!: "";
        @Prop() uploadLocalFuncSubPath!: "";
        @Prop() customRenderFunction!: null;
        @Prop() uploadFunc?: Function;
        @Prop() previewRenderFunc?: Function;

        private mde?: EasyMDE;

        @Watch("value", {immediate: true})
        onValueChange() {
            if (this.mde) {
                if (this.mde.value() != this.value) {
                    this.mde.value(this.value);
                }
            }
        }

        mounted() {
            // const toBase64 = (file: File) => new Promise((resolve, reject) => {
            //     const reader = new FileReader();
            //     reader.readAsDataURL(file);
            //     reader.onload = () => resolve(reader.result);
            //     reader.onerror = error => reject(error);
            // });

            let self = this;

            this.mde = new EasyMDE({
                element: (this.$refs.editor as HTMLElement),
                spellChecker: false,
                uploadImage: true,
                maxHeight: this.maxHeight,
                minHeight: this.minHeight,
                imageAccept: "image/png,image/jpeg,image/gif,docx,xlsx",
                async imageUploadFunction(file: any, onSuccess: any) {
                    if (self.uploadFunc != null) {
                        let data = await self.uploadFunc(file);
                        onSuccess(data.link);
                    }
                },
                previewRender: function(plainText: string) {
                    let result: string = "";
                    if (self.previewRenderFunc != null) {
                        result = self.previewRenderFunc(plainText);
                    } else {
                        result = marked(plainText);
                    }
                    return result; // Returns HTML from a custom parser
                },
            });
            if (this.mde) {
                this.mde.value(this.value);
                this.mde.codemirror.on("change", () => {
                    if (this.mde) {
                        this.$emit("input", this.mde.value())
                    }
                });
            }
        }

        beforeDestroy() {
            if (this.mde) {
                this.mde.toTextArea();
            }
            this.mde = undefined;
        }


    }
</script>

<style scoped>
    @import "~easymde/dist/easymde.min.css";
</style>

<style>
/*.CodeMirror,*/
/*.CodeMirror-scroll {*/
/*    max-height: 330px;*/
/*}*/

/*.CodeMirror-fullscreen .CodeMirror,*/
/*.CodeMirror-fullscreen .CodeMirror-scroll*/
/*{*/
/*    max-height: 100vh;*/
/*}*/
</style>