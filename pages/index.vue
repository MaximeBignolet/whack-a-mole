<template>
    <div
        class="h-screen w-screen bg-[url('/assets/images/bg_playground.png')] bg-no-repeat bg-cover bg-center overflow-hidden game_area">
        <img :src="cursoImg" :style="cursorStyle" class="custom-cursor" />
        <TransitionRoot appear :show="isOpen" as="template">
            <Dialog as="div" @close="closeModal" class="relative z-10">
                <TransitionChild as="template" enter="duration-300 ease-out" enter-from="opacity-0"
                    enter-to="opacity-100" leave="duration-200 ease-in" leave-from="opacity-100" leave-to="opacity-0">
                    <div class="fixed inset-0 bg-black/25" />
                </TransitionChild>
                <div class="fixed inset-0 overflow-y-auto">
                    <div class="flex min-h-full items-center justify-center p-4 text-center">
                        <TransitionChild as="template" enter="duration-300 ease-out" enter-from="opacity-0 scale-95"
                            enter-to="opacity-100 scale-100" leave="duration-200 ease-in"
                            leave-from="opacity-100 scale-100" leave-to="opacity-0 scale-95">
                            <DialogPanel
                                class="w-full max-w-md transform overflow-hidden rounded-3xl bg-white p-1.5 text-left align-middle shadow-xl transition-all">
                                <div class="bg-[#0A3442] p-4 rounded-3xl">
                                    <div class="bg-[#02799D] p-2 rounded-3xl">
                                        <DialogTitle as="h3" class="text-lg font-medium leading-6 text-gray-900">
                                            <div class="flex justify-center items-center gap-2">
                                                <img src="/assets/images/timer.png" alt="" class="h-20 w-20" />
                                            </div>
                                        </DialogTitle>
                                        <div class="flex flex-col items-center px-10 text-center">
                                            <div class="mt-4">
                                                <p class="text-white text-2xl font-bold">
                                                    Le temps est fini !
                                                </p>
                                            </div>
                                            <div class="mt-4">
                                                <p class="text-white text-xl font-bold">
                                                    Score final : {{ finalScore }}
                                                </p>
                                                <p v-if="record !== '0'" class="text-[#E17E1D] text-xl font-bold">
                                                    Record : {{ record }}
                                                </p>
                                                <p v-else>
                                                    Record : Pas encore de record personnel
                                                </p>
                                            </div>
                                            <div class="mt-4">
                                                <button type="button"
                                                    class="inline-flex justify-center rounded-lg mb-4 border border-transparent bg-[#E17E1D] px-10 py-2 text-sm font-medium text-white"
                                                    @click="{ closeModal(); startGame() }">
                                                    Rejouer
                                                </button>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </DialogPanel>
                        </TransitionChild>
                    </div>
                </div>
            </Dialog>
        </TransitionRoot>
        <div class="flex flex-col 2xl:gap-10 lg:gap-0 justify-center items-center container mx-auto pt-10">
            <div class="flex justify-center items-center gap-20 min-h-[130px]">
                <div class="bg-white rounded-full p-1 relative">
                    <img src="/assets/images/score_coin.png" alt="" class="h-14 w-14" />
                    <p
                        class="absolute bg-white rounded-md w-44 h-10 top-1/2 right-[92%] transform font-bold -translate-y-1/2 text-center text-3xl">
                        <span class="block pt-0.5">{{ score }}</span>
                    </p>
                </div>
                <div>
                    <img src="/assets/images/logo.png" alt="" class="h-32 w-32" />
                </div>
                <div class="relative">
                    <img src="/assets/images/timer.png" alt="" class="h-14 w-14" />
                    <div class="relative">
                        <p
                            class="absolute -top-7 left-[250%] w-44 h-10 rounded-md bg-white transform -translate-x-1/2 -translate-y-1/2 text-white text-2xl">
                        </p>
                        <p class="absolute -top-[46px] rounded-md left-[94%] w-[172px] h-9 bg-[#0A3442]"></p>
                        <p class="absolute -top-[46px] rounded-md left-[94%] w-[172px] h-9 bg-[#02799D]" ref="timeBar">
                        </p>
                    </div>
                </div>
            </div>
            <div class="grid grid-cols-3 place-items-start 2xl:gap-20 lg:gap-5 mb-10 lg:mt-0 2xl:mt-10 select-none">
                <div class="h-fit w-44 select-none" v-for="(mole, index) in moles" :key="index">
                    <img src="/assets/images/frame_6.svg" :id="'mole_' + index" class="mole"
                        @click="incrementScore(index)" />
                </div>
            </div>
            <p class="bg-[#42D3FF] py-2 px-6 shadow rounded-md text-xl font-bold" @click="startGame"
                v-if="!hasGameStarted">
                Commencer
            </p>
        </div>
    </div>
</template>

<script setup lang="ts">
import gsap from 'gsap';
import {
    TransitionRoot,
    TransitionChild,
    Dialog,
    DialogPanel,
    DialogTitle,
} from '@headlessui/vue'
import { ref, watchEffect, onMounted, onUnmounted, nextTick } from 'vue';
import { Howl } from 'howler';

const isOpen = ref(false);

function closeModal() {
    isOpen.value = false;
}

const randomIndex = ref(0);
const randomTime = ref(0);
const moles = Array.from({ length: 9 }, (_, index) => index);
const score = ref(0);
const hasClickedOnMole = ref(false);
const isGameOver = ref(true);
const timeLeft = ref(60);
const hasGameStarted = ref(false);
const cursoImg = "/assets/images/hammer_cursor.png";
const cursorStyle: Ref<Record<string, string>> = ref({
    position: 'absolute',
    top: '0',
    left: '0',
    width: '80px',
    height: '80px',
    transform: 'translate(-50%, -50%)',
});
let timeOutId: any;
let moleTimeoutId: any;
let timelines: any[] = [];
let timeLinesHit: any[] = [];
const finalScore = ref<string | null>('0');
const frame1 = "/assets/images/frame_1.svg";
const frame2 = "/assets/images/frame_2.svg";
const frame3 = "/assets/images/frame_3.svg";
const frame4 = "/assets/images/frame_4.svg";
const frame5 = "/assets/images/frame_5.svg";
const frame6 = "/assets/images/frame_6.svg";
const frameHit1 = "/assets/images/frame_hit_1.png";
const frameHit2 = "/assets/images/frame_hit_2.png";
const frameHit3 = "/assets/images/frame_hit_3.png";
const frameHit4 = "/assets/images/frame_hit_4.png";
const frames = [frame6, frame1, frame2, frame3, frame4, frame5, frame6];
const framesHit = [frameHit1, frameHit2, frameHit3, frameHit4, frame6];
const timeBar = ref();
const record = ref(localStorage.getItem('record') || '0');

const backgroundMusic = new Howl({
    src: '/assets/bg_music.mp3',
    loop: true,
    volume: 0.4,
});

const hit = new Howl({
    src: '/assets/hit.wav',
    volume: 0.5,
});

const updateCursorPosition = (event: MouseEvent) => {
    cursorStyle.value.top = `${event.clientY}px`;
    cursorStyle.value.left = `${event.clientX}px`;
};

function randomizeTimer() {
    randomTime.value = Math.floor(Math.random() * 1600) + 600;
}

function startRandomInterval() {
    randomizeTimer();
    timeOutId = setTimeout(startRandomInterval, randomTime.value);
}

function randomizeMoleVisibility() {
    randomIndex.value = Math.floor(Math.random() * moles.length);
    timelines.forEach((timeline, index) => {
        if (index === randomIndex.value) {
            timeline.play(0);
        } else {
            timeline.pause(0);
        }
    });
    moleTimeoutId = setTimeout(randomizeMoleVisibility, randomTime.value);
    hasClickedOnMole.value = false;
}

function incrementScore(index: number) {
    if (!hasGameStarted.value || index !== randomIndex.value || hasClickedOnMole.value) return;
    score.value += 10;
    hit.play();
    hasClickedOnMole.value = true;
    const timelineHit = gsap.timeline({ repeatDelay: 0.5 });
    framesHit.forEach((frame, frameIndex) => {
        timelineHit.to(
            `#mole_${index}`,
            {
                attr: { src: frame },
                duration: 0.2,
                ease: 'steps(1)',
            },
            frameIndex * 0.1
        );
    });
    timeLinesHit.push(timelineHit);
}

function animateMoles() {
    moles.forEach((_, index) => {
        const timeline = gsap.timeline({ repeatDelay: 0.5 });
        frames.forEach((frame, frameIndex) => {
            timeline.to(
                `#mole_${index}`,
                {
                    attr: { src: frame },
                    duration: 0.2,
                    ease: 'steps(1)',
                },
                frameIndex * 0.15
            );
        });
        timelines.push(timeline);
    });
}

function startGame() {
    startRandomInterval();
    randomizeMoleVisibility();
    backgroundMusic.play();
    if (timeBar.value) {
        timeBar.value.classList.add('timer-animation');
    }
    const timer = setInterval(() => {
        timeLeft.value -= 1;
        if (timeLeft.value <= 0) {
            endGame(timer);
        }
    }, 1000);
    isGameOver.value = false;
    animateMoles();
    hasGameStarted.value = true;
}

async function endGame(timer: any) {
    clearTimeout(timeOutId);
    clearTimeout(moleTimeoutId);
    backgroundMusic.stop();
    localStorage.setItem('score', score.value.toString());
    isGameOver.value = true;
    await nextTick();
    finalScore.value = score.value.toString();
    timeLeft.value = 60;
    isOpen.value = true;
    timeBar.value.classList.remove('timer-animation');
    score.value = 0;
    hasGameStarted.value = false;
    clearInterval(timer);
    updateRecord();
}

function updateRecord() {
    const currentRecord = parseInt(localStorage.getItem('record') || '0');
    if (parseInt(finalScore.value || '0') > currentRecord) {
        localStorage.setItem('record', finalScore.value || '0');
        record.value = finalScore.value || '0';
    } else {
        record.value = currentRecord.toString();
    }
}

watchEffect(() => {
    if (isGameOver.value) {
        finalScore.value = localStorage.getItem('score');
    }
});

onMounted(() => {
    document.addEventListener('mousemove', updateCursorPosition);
    document.body.style.cursor = 'none';
});

onUnmounted(() => {
    clearTimeout(timeOutId);
    clearTimeout(moleTimeoutId);
    document.removeEventListener('mousemove', updateCursorPosition);
});
</script>

<style scoped>
.custom-cursor {
    position: absolute;
    top: 0;
    left: 0;
    pointer-events: none;
    z-index: 1000;
}

@keyframes decreaseWidth {
    from {
        width: 172px;
    }

    to {
        width: 0px;
    }
}

.timer-animation {
    animation: decreaseWidth 60s linear forwards;
}

.mole {
    -webkit-user-drag: none;
}
</style>