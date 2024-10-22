<template>
    <div class="w-full h-screen bg-gray-800 p-4 text-gray-300 ">
        <div class="my-4">
            <h2>影片</h2>
            <label v-if="transcript.length" class="file-label"  @click="reset">重設</label>
            <label v-else for="myFile" class="file-label">選擇</label>
            <input ref="fileInput" id="myFile" type="file" @change="onFileChange" accept="video/*" class="file-input"  />   
        </div>
        <div class="flex flex-col-reverse md:flex-row w-full justify-center items-center border border-gray-600 shadow-md shadow-gray-500/50 rounded-md p-6 min-h-80">
            <div v-if="transcript.length" class="w-full md:w-2/5 max-w-80 pr-4">
                <div v-for="(item, i) in transcript" :key="item.title" class="my-2">
                    <h3 class="font-bold text-gray-100">{{item.title}}</h3>
                    <div v-for="(section, index) in item.highlight" :key="index"
                        :class="{'active':isSelectedHighlight(calcTime(i*transcript.length + index))}"
                        @click="selectHighlight({...section, timeline:calcTime(i*transcript.length + index)})"
                        class="text-sm cursor-pointer hover:text-blue-300">
                        <span class="text-blue-400 mr-2">{{ calcTime(i*transcript.length + index) }}</span>
                        <span>{{ section.content }}</span>
                    </div>
                </div>
            </div>
            <div v-else class="w-2/5 max-w-80 pr-4"></div>
            <div class="w-full md:w-3/5 max-w-[488px]">
                <video v-if="videoUrl" controls :src="videoUrl" @loadedmetadata="onMetadataLoaded"></video>
                <div v-else class="w-[400px] flex justify-center items-center h-60 border-2 border-dashed border-gray-400 bg-gray-200">
                    <span class="text-gray-600">Video</span>
                </div>
                <div class="flex justify-center">
                    <div v-if="transcript.length" class="w-full max-w-[460px] h-7 bg-gray-300 relative my-3">
                        <div v-for="(highlight, index) in selectedHighlights" :key="index" 
                            :style="{ 'left': `${Math.floor((calcSec(highlight.timeline) / videoDuration) * 100)}%` }" 
                            class="absolute bg-blue-500 h-7 w-3 rounded-full cursor-pointer"
                            @click="playHighlight(highlight)">
                        </div>
                    </div>
                </div>

            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import {ref} from 'vue' 

// 定義類型
interface Highlight {
    timeline: string;
    content: string;
}

interface TranscriptItem {
    title: string;
    highlight: Highlight[];
}

const videoUrl = ref('')
const transcript = ref<TranscriptItem[]>([])
const videoDuration= ref(0)
const selectedHighlights = ref([])

const onFileChange =(event: any)=> {
    const file = event.target.files[0];
    if (file && file.type.startsWith('video/')) {
        videoUrl.value = URL.createObjectURL(file); // 創建影片的 URL
        mockAIProcessing(); // 調用模擬 AI 處理
    } else {
        alert('請選擇一個有效的影片文件。');
    }
}
const onMetadataLoaded = (event: any )=> {
    videoDuration.value = event.target.duration; // 獲取影片的時長
}
const isSelectedHighlight = (t:string) => {
    return selectedHighlights.value.some((highlight:Highlight) => 
        highlight.timeline === t
    );
}
const mockAIProcessing = ()=>{
   
    // 模擬 API 返回的數據
    const mockResponse = [
            {
                title: "Introduction",
                highlight:[ 
                    {timeline:"",content:"Hello~~"},
                    {timeline:"",content:"Today,we'll be showcasing our latest innovation"}
                ]
            },
            {
                title: "Key Featurews",
                highlight:[ 
                    {timeline:"",content:"Our product has three main features"},
                    {timeline:"",content:"First..."},
                    {timeline:"",content:"Second..."},
                ]
            },
            {
                title: "Conclustion",
                highlight:[ 
                    {timeline:"",content:"In conclusion, our product is a game-changer"},
                    {timeline:"",content:"Thank you"}
                ]
            }
        ];

    // 模擬延遲以模擬 API 請求
    setTimeout(() => {
        transcript.value = mockResponse; // 將模擬數據賦值給 transcript
    }, 1000);
}
const calcTime = (i:number)=>{
    const section = Math.floor(videoDuration.value/11)
    const sectionSec = i < 2?section * (i + 1):i<6?section * (i + 2):section * (i + 3)
    
    const min = Math.floor(sectionSec / 60)
    const sec = sectionSec - (min * 60)
    return `${min}:${sec}`
}
const calcSec = (t:string)=>{
    const timeArr =t.split(':')
    console.log('t', t, timeArr)
    return Number(timeArr[0])*60 + Number(timeArr[1])
}
const selectHighlight = (section: Highlight)=> {
     // 檢查 selectedHighlights 中是否已存在該重點片段
    const exists = selectedHighlights.value.some(highlight => highlight.timeline === section.timeline && highlight.content === section.content);
    if (!exists) {
        // 將選擇的重點片段添加到 selectedHighlights
        selectedHighlights.value.push(section);
    }else{
        // 移除已存在的重點片段
        const index = selectedHighlights.value.findIndex(highlight => highlight.timeline === section.timeline && highlight.content === section.content);
        if (index !== -1) {
            selectedHighlights.value.splice(index, 1); // 移除該重點片段
        }
    }
}
const playHighlight = (section)=>{
    console.log('section', section)
    const videoElement = document.querySelector('video');
    if (videoElement) {
        videoElement.currentTime = calcSec(section.timeline); // 跳轉至指定時間
        videoElement.play(); // 播放影片
    }
}
const fileInput = ref(null)
const reset = ()=>{
    transcript.value = []
    videoUrl.value = ''
    videoDuration.value = 0
    selectedHighlights.value = []
    if (fileInput.value) {
        fileInput.value.value = null;
    }
}
</script>

<style scoped>
.file-input {
    display: none; /* 隐藏原始输入框 */
}

.file-label {
    display: inline-block;
    margin: 8px 0;
    padding: 4px 32px;
    background-color:rgb(42, 89, 137.5);
    color: white;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.3s;
}

.file-label:hover {
    background-color: #357ab8;
}
.active{
    background: rgba(64, 158, 255, 0.)
}
.custom-video-player {
    width: 100%; /* 使视频播放器宽度占满父容器 */
    background-color: #000; /* 设置播放器背景颜色 */
    border: 2px solid #4A90E2; /* 设置边框颜色 */
    border-radius: 10px; /* 设置圆角 */
}
</style>