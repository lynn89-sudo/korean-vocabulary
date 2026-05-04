<script>
    import { base } from "$app/paths";
    import { fade, fly, slide } from "svelte/transition";
    import { onMount } from "svelte";
    import { confetti } from '@neoconfetti/svelte';
    //import { passive } from "svelte/legacy"; <- This autoimported and I'm not sure why

    let korean = [
        [
            "빨간색",
            "Red"
        ],
        [
            "파란색",
            "Blue"
        ],
        [
            "분홍색",
            "Pink"
        ],
        [
            "검은색",
            "Black"
        ],
        [
            "하얀색",
            "White"
        ],
        [
            "곰",
            "Bear"
        ],
        [
            "새",
            "Bird"
        ],
        [
            "상표",
            "Sticker"
        ],
        [
            "우유",
            "Milk"
        ],
        [
            "토끼",
            "Rabbit"
        ],
        [
            "버섯",
            "Mushroom"
        ],
        [
            "대양",
            "Ocean"
        ],
        [
            "늘",
            "Always"
        ],
        [
            "강",
            "River"
        ],
        [
            "꽃",
            "Flower"
        ],
        [
            "물고기",
            "Fish"
        ],
        [
            "불",
            "Fire"
        ],
        [
            "벨",
            "Bell"
        ],
        [
            "모자",
            "Hat"
        ],
        [
            "사과",
            "Apple"
        ],
        [
            "배",
            "Pear"
        ],
        [
            "망고",
            "Mango"
        ],
        [
            "주스",
            "Juice"
        ],
        [
            "올빼미",
            "Owl"
        ],
        [
            "달",
            "Moon"
        ],
        [
            "밤",
            "Night"
        ],
        [
            "시계",
            "Clock"
        ],
        [
            "컴퓨터",
            "Computer"
        ],
        [
            "자전거",
            "Bicycle"
        ],
        [
            "한국인",
            "Korean"
        ]
    ]


    let assessed = [];
    for (let i = 0;  i < korean.length; i++) {
        assessed.push(i);
    }

    let gameStatus = $state(0);
    let answerStatus = $state(0);

    let round = $state({
        "curr": "",
        "corr": "",
        "choices": []
    });

    function generateRound() {
        if (assessed.length == 0) {
            for (let i = 0;  i < korean.length; i++) {
                assessed.push(i);
            }
        }
        round.choices = [];
        let rand = Math.floor((Math.random() * assessed.length));
        rand = assessed[rand];
        assessed = assessed.filter((index) => index != rand);
        round.curr = korean[rand][0];
        round.corr = korean[rand][1];

        let corrIndex = Math.floor(Math.random() * 4);
        for (let i = 0; i < 4; i++) {
            if (corrIndex == i) {
                round.choices.push(round.corr);
            }
            else {
                let rand2 = Math.floor((Math.random() * korean.length));
                if (rand2 == rand) {
                    i--;
                    continue;
                }
                else if (checkChoices(korean[rand2][1])) {
                    round.choices.push(korean[rand2][1]);
                }
                else {
                    i--; 
                }
            }
        }
        //console.log(round.choices);
        totalQuestions++;
    }

    function checkChoices(choice) {
        for (let i = 0; i < round.choices.length; i++) {
            if (round.choices[i] == choice) {
                return false;
            }
        }
        return true;
    }

    let correctEmote = $state(true);
    function progress (correct) {
        setTimeout(function() {answerStatus = 1}, 500);
        if (correct) {
            setTimeout(function() {streak++; correctAnswers++;}, 500);
            setTimeout(()=> {if (gameStatus != 3) {answerStatus = 0; gameStatus = 2; generateRound();}}, 2500);
            setTimeout(function() {if (gameStatus != 3) {gameStatus = 1}}, 3000);
            setTimeout(function() {
                if (streak > maxStreak) {
                    maxStreak = streak;
                    //console.log("Max Streak", maxStreak);
                }
            }, 600)
        }
        else {
            setTimeout(function() {correctEmote = false}, 500)
            setTimeout(function() {streak = 0}, 1000)
            //console.log(correct);
            setTimeout(()=> {if (gameStatus != 3) { answerStatus = 0; gameStatus = 2; generateRound() }}, 3500);
            setTimeout(function() {if (gameStatus != 3) { gameStatus = 1; correctEmote = true }}, 4000);
            streak = 0;
        }
    }

    onMount(() => {
        generateRound();
        stat();
    })

    function stat() {
        let secondPer = 15;
        let prepost = 15;
        console.log(korean.length + " words are on this game. If one question is completed every 15 seconds, there is an about " + (Math.floor(((420/15)/korean.length)*100) + "% chance each word gets quizzed in the game"));
        console.log(Math.floor((prepost/korean.length)*100) + "% is tested on pre-post")
    }

    let timerMode = $state(0);
    let timer = $state(420); // 7 minutes (60*7) > Set to 420
    let timerString = $state(convertTimerString());

    let timerBreak;
    /*
    onMount(() => {
        timerBreak = window.setInterval(timerCount, 1000);
    })
    */
   let triggerConfetti = $state(false);
    function timerCount() {
        if (timerMode == 1) {
            timerString = convertTimerString();
            //console.log(timer);
            timer--;
        }
        if (timer < -1) {
            timerMode = 0;
            gameStatus = 3;
            correctEmote = 1;
            console.log(totalQuestions + " questions completed");
            window.clearInterval(timerBreak);
            if (correctAnswers > 0) {
                window.setTimeout(() => {triggerConfetti = true}, 1500);
            }
        }
    }
    function convertTimerString() {
        let mins = Math.floor(timer/60);
        let secs = timer%60;
        if (secs <= 9) {
            secs = "0" + secs;
        }
        let str = mins + ":" + secs;
        return str;
    }

    let correctAnswers = $state(0);
    let streak = $state(0);
    let maxStreak = $state(0);

    // Testing
    let totalQuestions = $state(0);

</script>
<svelte:head>
    <link rel="preload" as="image" href="{base}/images/imke.png"/>
    <link rel="preload" as="image" href="{base}/images/imkeAngry.png"/>
    <title>Korean Vocabulary Game</title>
</svelte:head>
<style>
    #container {
        position: fixed;
        transform: translate(-50%, -50%);
        transition: top 0.5s ease-in-out;
        left: 50%;
        top: 50%;
        padding: 20px;
        background-color: rgb(136, 61, 33);
        box-shadow: 0px 0px 20px 10px rgba(127, 91, 24, 0.544);
        border-radius: 20px;
        width: 90%;
        height: 78.5%;
        z-index: 999;
    }
    #container.active {
        top: 53.5%;
    }
    #bird {
        z-index: 1000;
        position: fixed;
        transform: translate(-50%, 0%);
        left: 50%;
        bottom: -110px;
        animation: pulse 2s infinite ease-in-out;
        
        img {
            max-width: 280px;
            transition: transform 0.3s ease-in-out;
        }
    }
    @keyframes pulse {
        0%, 100% {
            bottom: -105px;
        }
        50% {
            bottom: -115px;
        }
    }

    button {
        background-color: white;
        color: rgb(136, 61, 33);
    }

    #choices {
        display: flex;
        justify-content: center;
        button {
            padding: 20px;
            margin: 8px;
            user-select: none;
        }
        button:hover {
            transform: scale(1.05);
        }
        button.correct {
            background-color: lightgreen;
            color: rgb(53, 99, 53);
            transform: scale(1);
        }
        button.incorrect {
            background-color: lightcoral;
            color: rgb(92, 49, 49);
            transform: scale(1);
        }
        button.pauseMode {
            transform: scale(1);
        }
    }

    button.pause {
        padding: 5px;
        margin: 5px;
        transform: translateY(3px);
        span {
            transform: translateY(2px)
        }
        border-radius: 30px;
    }

    #streak {
        position: fixed;
        transform: translate(-50%, 2%);
        left: 50%;
        background-color: rgb(75, 33, 17);
        padding: 0px 20px;
        border-radius: 20px;
        h3 {
            transform: translateY(-3.5px);

            span {
                transform: translateY(3.5px);
            }
        }
    }

    .confetti {
        position: fixed;
        top: 10%;
        left: 50%;
        transform: translate(-50%, -50%);
        pointer-events: none;
        z-index: 999;
    }
</style>
{#if gameStatus == 1 || gameStatus == 2}
<div id="streak" in:slide={{delay: 700}} out:slide>
    <h3><span class="material-symbols-outlined">check_circle</span> {correctAnswers} <span class="material-symbols-outlined" translate="no">mode_heat</span> {streak}</h3>
</div>
{/if}
<div id="container" class:active={gameStatus == 1 || gameStatus == 2}>
    {#if gameStatus == 1 || gameStatus == 2}
    <div style:margin-top=5px in:fade={{delay: 500}} out:fade>
        <p style:font-weight=800>{timerString} <button class="pause" onclick={() => {if (timerMode == 1) { timerMode = 0} else {timerMode = 1}}}><span class="material-symbols-outlined" translate="no">{#if timerMode}pause_circle{:else}play_circle{/if}</span></button></p>
    </div>
    {/if}
    <div style:margin-top=20px>
        {#if gameStatus == 0}
        <div out:fade style:margin-top=60px>
            <h1>Korean Vocabulary Game</h1>
            <p>Let's learn Korean! Don't get questions wrong, or the bird gets angry...</p>
            <p><button onclick={function() {gameStatus = 0.5; setTimeout(() => {timerBreak = window.setInterval(timerCount, 1000); gameStatus = 1; timerMode = 1}, 500)}}>Start</button></p>
        </div>
        {/if}
        {#if gameStatus == 1}
        <div style:text-align="center" in:fly={{delay: 1000, y:100}} out:fade>
            <h1 translate="no" style:user-select="none" style:font-size=40px>{round.curr}</h1>
            <h2 style:color="white">Translate this word into its English equivalent</h2>
            <div id="choices">
                {#each round.choices as x}
                    {#if x == round.corr}
                    <button class:pauseMode={!timerMode} class:correct={answerStatus == 1} disabled={answerStatus == 1 || timerMode == 0} onclick={() => {progress(true)}}>{x}</button>
                    {:else}
                    <button class:pauseMode={!timerMode} class:incorrect={answerStatus == 1} disabled={answerStatus == 1 || timerMode == 0} onclick={() => {progress(false)}}>{x}</button>
                    {/if}
                {/each}
            </div>
        </div>
        {/if}
        {#if gameStatus == 3}
        <div style:margin-top=60px transition:fly={{delay: 500, y:200}}>
            {#if correctAnswers == 0}
            <h1>You'll do better next time</h1>
            {:else}
            <h1>Great Job!</h1>
            {/if}
            <h2 style="color: white; text-align: center;">Your highest streak was {maxStreak} <span style:transform="translateY(3px)" class="material-symbols-outlined">mode_heat</span></h2>
            {#if correctAnswers != 1}
            <p>{correctAnswers} correct answers <span class="material-symbols-outlined" style:transform="translateY(7.5px)">check_circle</span></p>
            {:else}
            <p>{correctAnswers} correct answer <span class="material-symbols-outlined" style:transform="translateY(8px)">check_circle</span></p>
            {/if}
        </div>
        {/if}
    </div>
</div>

{#if triggerConfetti == true}
<div class="confetti" use:confetti></div>
{/if}

<div id="bird">
    {#if correctEmote}
    <img src="{base}/images/imke.png" alt="Bird"/>
    {:else}
    <img src="{base}/images/imkeAngry.png" alt="Bird"/>
    {/if}
</div>