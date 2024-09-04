<script setup>
import {ref, onMounted} from 'vue'
import Editor from '@toast-ui/editor'
import '@toast-ui/editor/dist/toastui-editor.css'
import '@toast-ui/editor/dist/theme/toastui-editor-dark.css'
import axios from 'axios'

const props = defineProps({
  modelValue: {
    type: String,
    required: false,
    default: '',
  },
});
const emit = defineEmits(['update:modelValue', 'fileChange'])
const editor = ref()

function blobToBase64(blob) {
  return new Promise((resolve, _) => {
    const reader = new FileReader()
    reader.onloadend = () => resolve(reader.result)
    reader.readAsDataURL(blob)
  })
}

onMounted(() => {
  const e = new Editor({
    el: editor.value,
    height: '300px',
    initialEditType: 'markdown',
    previewStyle: 'tab',
    theme: 'dark',
    placeholder: 'Please paste image or enter text.',
    toolbarItems: [],
    events: {
      change: () => emit('update:modelValue', e.getMarkdown()),
    },
    hooks: {
      addImageBlobHook: async (blob, callback) => {
        const formData = new FormData()
        formData.append('image', blob)
        callback(await blobToBase64(blob))

        // try {
        //     const response = await axios.post('/upload', formData, {
        //         headers: {
        //             'Content-Type': 'multipart/form-data'
        //         }
        //     });
        //     const imageUrl = response.data.url; // 假设后端返回图片的URL

        //     // 使用回调函数将图片URL插入Markdown内容中
        //     callback(imageUrl, 'image alt text');
        // } catch (error) {
        //     console.error('Image upload failed:', error);
        // }
      }
    },
  });
  e.addCommand('markdown', 'text2md', (...params) => {
    emit('fileChange', text2Md(e.getMarkdown()))
    // const cm = mde.getEditor();  // 获取CodeMirror实例
    // cm.replaceSelection('**这是自定义插入的文本**');
  })
  e.insertToolbarItem({ groupIndex: 0, itemIndex: 0 }, {
    name: 'md',
    tooltip: '转为md文件',
    text: '完成',
    command: 'text2md',
    className: 'toastui-editor-toolbar-icons first',
    style: { backgroundImage: 'none', color: '#fff' }
  });
});

function text2Md(text, filename = 'temp.md') {
  const blob = new Blob(['\ufeff', text], {
    type: "text/plain",
  });

  const file = new File([blob], filename, {type: "text/plain"})
  const fileURL = URL.createObjectURL(file)
  return file
}
</script>

<template>
  <div>
    <div ref="editor"></div>
  </div>
</template>
<style>
.toastui-editor-defaultUI-toolbar {
  justify-content: flex-end;
}
</style>
