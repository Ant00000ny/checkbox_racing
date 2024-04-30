<script setup>
import anime from "animejs";
import moment from "moment-timezone";
import {useIntervalFn} from "@vueuse/core";

useHead({
    title: 'Checkbox Racing',
    bodyAttrs: {
        class: 'h-screen w-screen overflow-hidden m-0 p-0'
    }
})

function translateWrapperToCurrent() {
    document.querySelectorAll('.wrapper')[0].style.transform = 'translateX(' + currentTranslateXPx + 'px)';
}

function calculateDistanceBetweenEl(element1, element2) {
    // Get bounding rectangles of both elements
    const rect1 = element1.getBoundingClientRect();
    const rect2 = element2.getBoundingClientRect();

    // Calculate the distance between points
    return Math.abs(rect1.left - rect2.left);
}
function handleClickOutsideBox() {
    if (!started) return
    if (currentBoxIndexToBeClicked === 0) return
    currentBoxIndexToBeClicked--;
    checkBoxesRef.value[currentBoxIndexToBeClicked + 1].enabled = false
    checkBoxesRef.value[currentBoxIndexToBeClicked].enabled = true
    checkBoxesRef.value[currentBoxIndexToBeClicked].checked = false

    currentTranslateXPx += checkBoxDistance
    translateWrapperToCurrent()

}

function handleClickCheckBox() {
    currentBoxIndexToBeClicked++;
    checkBoxesRef.value[currentBoxIndexToBeClicked - 1].enabled = false
    console.log(checkBoxesRef.value.length)
    console.log(currentBoxIndexToBeClicked)
    if (checkBoxesRef.value.length <= currentBoxIndexToBeClicked) {
        handleWin()
        return
    }
    checkBoxesRef.value[currentBoxIndexToBeClicked].enabled = true
    // random translateY
    let factor = 0.3 + 0.7 * currentBoxIndexToBeClicked / checkBoxesRef.value.length
    document.querySelectorAll('input[type="checkbox"]')[currentBoxIndexToBeClicked].style.transform = 'translateY(' + (Math.floor(Math.random() * 200) - 100) * factor + 'px)'
    currentTranslateXPx -= checkBoxDistance
    translateWrapperToCurrent()
}

function handleWin() {
    timer.pause()
    anime({
        targets: '.timer',
        top: ['50%', '20%'],
        opacity: '1',
        duration: 1000,
        easing: 'easeInOutQuad',
        complete: () => {
            alert(`You win! Avg speed: ${(currentBoxIndexToBeClicked / (Date.now() - startTime) * 1000).toFixed(3)} checks/s`)
        }
    });
    started = false
}

let timer;
let started = false
function handleStart() {
    if (started) {
        return
    }
    started = true
    timer = useIntervalFn(() => {
        timeElapsed.value = moment.utc(Date.now() - startTime).format('mm:ss:SS')
    }, 10);
    startTime = Date.now()
}

const checkBoxesRef = ref(Array.from({length: 50}, () => ({checked: false, enabled: false})));
let currentBoxIndexToBeClicked = 0;
let currentTranslateXPx = 0;
let checkBoxDistance;
let startTime;
let timeElapsed = ref('00:00:00');


onMounted(() => {
    checkBoxesRef.value[currentBoxIndexToBeClicked].enabled = true

    const checkBoxes = document.querySelectorAll('input[type="checkbox"]');
    checkBoxDistance = calculateDistanceBetweenEl(checkBoxes[0], checkBoxes[1])

});

</script>

<template>
    <div class="timer fixed top-1/2 left-1/2 -translate-y-1/2 -translate-x-1/2 text-[10vw] font-mono opacity-[0.02] pointer-events-none select-none">{{ timeElapsed }}</div>
    <div @click="handleClickOutsideBox" class="root">
        <div class="wrapper">
            <div>
                <div class="fixed -translate-x-full -translate-y-1/2 flex justify-center items-center space-x-2">
                    <span class="text-nowrap font-thin align-baseline select-none pointer-events-none">Check this to start</span>
                    <Icon name="material-symbols-light:arrow-forward-rounded"/>
                </div>
            </div>
            <input type="checkbox"
                   class="checkbox"
                   @click.stop="handleClickCheckBox();index === 0 ? handleStart() : ''"
                   @keydown.space.prevent
                   @keydown.tab.prevent
                   v-model="item.checked"
                   :disabled="!item.enabled"
                   v-for="(item, index) in checkBoxesRef"/>
            <Icon name="material-symbols-light:flag" color="red"/>
        </div>
    </div>
</template>

<style scoped>
* {
    box-sizing: border-box;
}

.root {
    @apply h-screen w-screen;
    @apply flex ;
}

.wrapper {
    @apply w-screen ml-[50vw];
    @apply flex items-center justify-start space-x-2;
    @apply transition-transform;
}

.checkbox {
    @apply transition-transform duration-500;
}

input:disabled {
    @apply cursor-not-allowed;
}
</style>
