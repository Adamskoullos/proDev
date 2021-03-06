<template>
    <div class="chat-window">
        <div v-if="error" class="error">{{ error }}</div>
        <div v-if="documents" class="messages" ref="messages">
            <transition-group name="list" appear>
                <div v-for="doc in formattedDocuments" :key="doc.id" class="single" :class="{light: light}">
                    <div class="timestamp" :class="{light: light}">
                        <span class="created-at" :class="{light: light}">{{ doc.createdAt }}</span>
                        <span class="name" :class="{light: light}">{{ doc.name }}</span>
                    </div>
                    <span class="message" :class="{light: light}">{{ doc.message }}</span>
                </div>
            </transition-group>
        </div>
    </div>
</template>

<script>
import getCollectionMessages from '../composables/getCollectionMessages'
import { formatDistanceToNow } from 'date-fns'
import { computed, onUpdated, ref } from '@vue/runtime-core'


export default {
    props: ['light'],
    setup(){
        const { documents, error } = getCollectionMessages('messages')

        const formattedDocuments = computed (() => {
            if(documents.value){
                return documents.value.map(doc => {
                    // The below `formatDistanceToNow` is a method from the date-fns library
                    let time = formatDistanceToNow(doc.createdAt.toDate())
                    return { ...doc, createdAt: time }
                })
            }
        })

        // auto-scroll to bottom of messages to show the newest
        const messages = ref(null)
        // fire on every update to maintain always showing the latest messages
        // The scrollTop/scrollHeight trick was obtained from the Vue 3 course detailed within the credits of the readme
        onUpdated(() => {
            messages.value.scrollTop = messages.value.scrollHeight
        })

        return { formattedDocuments, documents, error, messages }
    }
}
</script>

<style scoped>
.chat-window{
    display: flex;
    flex-direction: column;
    align-items: stretch;
    justify-content: flex-end;
}
.messages{
    max-height: 70vh;
    box-sizing: border-box;
    overflow:auto;
}
/* Hiding the scrollbar to maintain a clean look */
/* The below scrollbar properties we made aware to me from this CSS Tricks linK: https://css-tricks.com/custom-scrollbars-in-webkit/ */
.messages::-webkit-scrollbar {
    width: 0px;               /* width of the entire scrollbar */
}

.messages::-webkit-scrollbar-track {
background: rgb(75,75,75);        /* color of the tracking area */
}

.single{
    background: rgb(63, 63, 63, 0.9);
    border-radius: 8px;
    min-height: 50px;
    display: flex;
    flex-direction: column;
    align-items: stretch;
    justify-content: space-between;
    margin: 10px auto;
}
.single.light{
    color: var(--primary);
    background: rgba(255, 255, 255, 0.9);
}
.single span.light{
    color: var(--primary);
}
.timestamp span.light{
    color: var(--primary);
}
.timestamp{
    flex:1;
    display: flex;
    justify-content: space-between;
}
.timestamp span{
    font-weight: 200;
    margin: 5px 15px 0 15px;
}
span.message{
    line-height: 1.1rem;
    margin: 10px 15px;
}
/* Transitions for new messages as well as loading message elements */

.list-enter-from,
.list-leave-to{
  opacity: 0;
  transform: scale(0.1);
}

.list-enter-to,
.list-leave-from{
  opacity: 1;
  transform: scale(1);
}
.list-leave-active{
   transition: all 0.5s ease; 
}
.list-enter-active{
  transition: all 0.7s ease;
}
.list-move{
    transition: all 0.3s ease;
}
</style>