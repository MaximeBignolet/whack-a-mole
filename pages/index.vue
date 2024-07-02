<template>
    <div
        class="h-screen w-screen bg-[url('/assets/images/bg_playground.png')] bg-no-repeat bg-cover bg-center overflow-hidden game_area">
        <TransitionRoot appear :show="showRankingsFlag" as="template">
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
                                                <div v-if="players.length > 1">
                                                    <p class="text-white text-xl font-bold">
                                                        Classement final : 
                                                    </p>
                                                    <ul>
                                                        <li v-for="(player, index) in sortedPlayers" :key="index">
                                                            <div class="flex items-center gap-3 text-[#E17E1D] font-bold my-5" :class="index === 0 ? 'text-2xl' : index === 1 ? 'text-lg' : 'text-md'">
                                                                <img src="/assets/images/couronne.png" alt="" v-if="index === 0" class="h-10">
                                                                <img src="/assets/images/silver_medal.svg" alt="" v-if="index === 1" class="h-8 my-5">
                                                                <img src="/assets/images/bronze_medal.svg" alt="" v-if="index === 2" class="h-8">
                                                                <p> {{ player.name }} - {{ player.score }} points</p>
                                                            </div>
                                                        </li>
                                                    </ul>
                                                </div>
                                                <p class="text-2xl font-bold text-white mb-5" v-if="players.length === 1">Le temps est écoulé !</p>
                                                <p v-if="players.length === 1" class="mb-3 font-bold text-lg text-white">
                                                    Score Final : {{ finalScore }}
                                                </p>
                                                <p v-if="record !== '0' && players.length === 1"
                                                    class="text-[#E17E1D] text-xl font-bold mb-5">
                                                    Record : {{ record }} (détenu par {{ playerWhoHaveRecord }})
                                                </p>
                                                <p class="py-2 px-3 mb-5 cursor-none rounded-lg bg-[#E17E1D] text-white font-bold text-lg" v-if="players.length === 1" @click="() => {showRankingsFlag = false; players.pop()}">
                                                    Revenir à l'accueil
                                                </p>
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
            <div class="grid grid-cols-4 grid-rows-3 place-items-start 2xl:gap-12 lg:gap-5 mb-10 lg:mt-0 2xl:mt-0 select-none cursor-none overflow-hidden"
                v-if="players.length > 0">
                <img :src="cursoImg" :style="cursorStyle" class="custom-cursor" />
                <div class="h-fit w-44 select-none" v-for="(mole, index) in moles" :key="index">
                    <img src="/assets/images/frame_6.svg" :id="'mole_' + index" class="mole"
                        @click="incrementScore(index)" />
                </div>
            </div>
            <div v-else class="bg-white rounded-3xl p-1 my-10">
                <div class="bg-[#0A3442] p-2 rounded-3xl">
                    <div class="bg-[#02799D] p-4 rounded-3xl">
                        <p class="text-white text-2xl font-bold text-center mt-3">Veuillez choisir le nombre de joueur
                        </p>
                        <div class="flex justify-center gap-3 items-center my-5">
                            <p class="bg-[#E17E1D] aspect-square cursor-pointer h-10 flex flex-col items-center text-white rounded-xl justify-center"
                                id="un" @click="onClickLogClass">1</p>
                            <p class="bg-[#E17E1D] aspect-square cursor-pointer h-10 flex flex-col items-center text-white rounded-xl justify-center "
                                id="deux" @click="onClickLogClass">2</p>
                            <p class="bg-[#E17E1D] aspect-square cursor-pointer h-10 flex flex-col items-center text-white rounded-xl justify-center trois"
                                id="trois" @click="onClickLogClass">3</p>
                            <p class="bg-[#E17E1D] aspect-square cursor-pointer h-10 flex flex-col items-center text-white rounded-xl justify-center quatre"
                                id="quatre" @click="onClickLogClass">4</p>
                        </div>
                        <div v-if="numberOfPlayers.length > 0">
                            <p class="text-white text-2xl font-bold text-center">Veuillez entrer votre nom pour
                                enregistrer votre score</p>
                            <form @submit.prevent="handleSubmit">
                                <div v-for="(player, index) in numberOfPlayers" :key="index" class="my-5">
                                    <div class="flex flex-col items-center gap-5">
                                        <label :for="'nom_joueur_' + index" class="text-white font-bold text-xl">Joueur
                                            {{ index + 1 }}</label>
                                        <input :id="'nom_joueur_' + index" type="text"
                                            placeholder="Entrez le nom du joueur" v-model="player.name"
                                            required
                                            class="py-2 px-5 rounded-xl placeholder:text-gray-300 placeholder:font-light text-white font-bold bg-[#E17E1D]" />
                                    </div>
                                </div>
                                <div class="flex justify-center mt-10">
                                    <button type="submit"
                                        class="inline-flex justify-center rounded-lg mb-4 border border-transparent bg-white px-5 py-2 text-sm font-medium ">Commencer
                                        la partie</button>
                                </div>
                            </form>
                        </div>
                    </div>
                </div>
            </div>
            <p class="bg-[#42D3FF] py-2 px-6 shadow rounded-md text-xl font-bold cursor-none" @click="startGame"
                v-if="!hasGameStarted && players.length">
                {{ players[currentPlayerIndex].name }} - Commence la partie
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

interface Player {
    name: string
    score: number
}

const randomIndex = ref(0);
const randomTime = ref(0);
const moles = Array.from({ length: 16 }, (_, index) => index);
const score = ref(0);
const hasClickedOnMole = ref(false);
const isGameOver = ref(true);
const timeLeft = ref(45);
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
const playerWhoHaveRecord = ref(localStorage.getItem('player') || '0');
const numberOfPlayers = ref<Player[]>([]);
const players = ref<Player[]>([]);
const isOpen = ref(false);
const currentPlayerIndex = ref(0);
const showRankingsFlag = ref(false);

function closeModal() {
    isOpen.value = false;
}

const onClickLogClass = (event: any) => {
    switch (event.target.id) {
        case 'un':
            numberOfPlayers.value = [{ name: '', score: 0 }];
            break;
        case 'deux':
            numberOfPlayers.value = [{ name: '', score: 0 }, { name: '', score: 0 }];
            break;
        case 'trois':
            numberOfPlayers.value = [{ name: '', score: 0 }, { name: '', score: 0 }, { name: '', score: 0 }];
            break;
        case 'quatre':
            numberOfPlayers.value = [{ name: '', score: 0 }, { name: '', score: 0 }, { name: '', score: 0 }, { name: '', score: 0 }];
            break;
    }
}

const handleSubmit = () => {
    players.value.push(...numberOfPlayers.value);
}


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
                duration: 0.15,
                ease: 'steps(1)',
            },
            frameIndex * 0.1
        );
    });
    timeLinesHit.push(timelineHit);
}

function animateMoles() {
    moles.forEach((_, index) => {
        const mole = document.querySelectorAll(`.mole`);
        const timeline = gsap.timeline({ repeatDelay: 0.5 });
        frames.forEach((frame, frameIndex) => {
        
            if(frame === '/assets/images/frame_5.svg' || frame === '/assets/images/frame_6.svg'){
                mole[5].classList.add('pointer-events-none')
                mole[6].classList.add('pointer-events-none')
            }
            timeline.to(
                `#mole_${index}`,
                {
                    attr: { src: frame },
                    duration: 0.15,
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
    timeLeft.value = 45;
    isOpen.value = true;
    timeBar.value.classList.remove('timer-animation');
    score.value = 0;
    clearInterval(timer);
    updateRecord();
    players.value[currentPlayerIndex.value].score = Number(finalScore.value);
    currentPlayerIndex.value += 1
    if (currentPlayerIndex.value === players.value.length) {
        showRankings();
        currentPlayerIndex.value = 0;
    }
    hasGameStarted.value = false;
}

function showRankings() {
    showRankingsFlag.value = true;
}

const sortedPlayers = computed(() => {
    return players.value.slice().sort((a, b) => b.score - a.score);
});


function updateRecord() {
    const currentRecord = parseInt(localStorage.getItem('record') || '0');
    const currentPlayerWhoHaveRecord = localStorage.getItem('player') || '0';
    if (parseInt(finalScore.value || '0') > currentRecord) {
        localStorage.setItem('record', finalScore.value || '0');
        localStorage.setItem('player', players.value[currentPlayerIndex.value].name || '0');
        record.value = finalScore.value || '0';
        playerWhoHaveRecord.value = players.value[currentPlayerIndex.value].name || '0';
    } else {
        record.value = currentRecord.toString();
        playerWhoHaveRecord.value = currentPlayerWhoHaveRecord.toString();    
    }
}

watchEffect(() => {
    if (isGameOver.value && playerWhoHaveRecord.value !== null) {
        finalScore.value = localStorage.getItem('score');
        playerWhoHaveRecord.value = localStorage.getItem('player') as string;
    }
});

onMounted(() => {
    document.addEventListener('mousemove', updateCursorPosition);

});

onUnmounted(() => {
    clearTimeout(timeOutId);
    clearTimeout(moleTimeoutId);
    document.removeEventListener('mousemove', updateCursorPosition);
});
</script>

<style scoped>

body {
    overflow: hidden;
}

.custom-cursor {
    position: absolute;
    top: 0;
    left: 0;
    pointer-events: none;
    z-index: 1000;
    overflow: hidden
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
    animation: decreaseWidth 45s linear forwards;
}

.mole {
    -webkit-user-drag: none;
    max-height: 130px;
}
</style>