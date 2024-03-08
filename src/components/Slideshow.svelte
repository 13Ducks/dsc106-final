<script>
    import { onMount } from "svelte";
    import Game from "./Game.svelte";
    import Simulation from "./Simulation.svelte";

    const NUM_GAMES = 10;

    let pastScores = { Cooperate: 0, Defect: 0, TFT: 0, Pavlov: 0 };
    let currentScore = 0;
    let gamesPlayedOnCurrentSlide = 0;
    let simDone = false;
    let disableForward = false;
    let disableBackward = false;

    const handleGamePlayed = (event) => {
        let data = event.detail;
        gamesPlayedOnCurrentSlide = data["playCount"];
        currentScore = data["userPayoff"];
        pastScore[data["opponentStrategy"]] = data["userPayoff"];
    };

    let slides;

    $: slides = [
        {
            content: `<p>Have you ever heard about the Prisoner's Dilemma?</p>`,
        },
        {
            content: `<p>It is a classic problem in game theory which pits two players against each other. 
                If Player A chooses to testify against Player B, and B stays silent, A gets out free while B gets 3 years in prison, and vice versa. If they both testify, they both serve 2 years. Finally, if they both stay silent, they each serve 1 year.`,
        },
        {
            content: `<p>What would you do?</p>`,
        },
        {
            content: `
            <p>If you answered testify, you're correct! (and a little heartless).</p>
      <svg width="400" height="400">
        <!-- Background -->

        <rect x="50" y="50" width="150" height="150" fill="#00ff00" stroke="black" stroke-width="2"/>
        <rect x="200" y="50" width="150" height="150" fill="#f0f0f0" stroke="black" stroke-width="2"/>
        <rect x="50" y="200" width="150" height="150" fill="#00ff00" stroke="black" stroke-width="2"/>
        <rect x="200" y="200" width="150" height="150" fill="#f0f0f0" stroke="black" stroke-width="2"/>
        
        <text x="80" y="30" font-size="20" fill="#000000">You Testify</text>
        <text x="220" y="30" font-size="20" fill="#000000">You're Silent</text>
        <text font-size="20" fill="#000000" transform="translate(30, 170) rotate(270)">They Testify</text>
        <text font-size="20" fill="#000000" transform="translate(30, 340) rotate(270)">They're Silent</text>

        <text x="95" y="130" font-size="36" fill="#000000">2, 2</text>
        <text x="245" y="130" font-size="36" fill="#000000">3, 0</text>
        <text x="95" y="285" font-size="36" fill="#000000">0, 3</text>
        <text x="245" y="285" font-size="36" fill="#000000">1, 1</text>

      </svg>
      <p>If they were to stay silent, you would get off free by testifying, and if they testified, you would get a lower sentence by testifying as well. Thus, the correct strategy is to always testify.</p>
    `,
        },
        {
            content: `<p>But what if you were playing multiple times...</p>`,
        },
        {
            content: `<p>You'll play 10 times against 4 imaginary opponents with different playstyles. Let's see how you do! For simplicity, we'll change the terminology (testify = defect and silent = cooperate) and reverse the game so a bigger number is better (defecting while the other cooperates is +3)</p>`,
        },
        { component: Game, props: { opponentStrategy: "Cooperate" } },
        {
            content: `<p>You scored ${currentScore}! Onto the next...</p>`,
        },
        { component: Game, props: { opponentStrategy: "Defect" } },
        {
            content: `<p>You scored ${currentScore}! Onto the next...</p>`,
        },
        { component: Game, props: { opponentStrategy: "TFT" } },
        {
            content: `<p>You scored ${currentScore}! Onto the next...</p>`,
        },
        { component: Game, props: { opponentStrategy: "Pavlov" } },
        {
            content: `<p>You scored ${currentScore}! Now to see the overall results...</p>`,
        },
        {
            content: `<p>Against the Always Cooperate bot, you scored ${pastScores["Cooperate"]}.</p>
                      <p>Against the Always Defect bot, you scored ${pastScores["Defect"]}.</p>
                      <p>Against the Tit for Tat bot, you scored ${pastScores["TFT"]}.</p>
                      <p>Against the Pavlov bot, you scored ${pastScores["Pavlov"]}.</p>
                      <br/>
                      <p>While defecting will always mean you don't <i>lose</i>, it is possible to score more points overall by cooperating. For example, if you always cooperate against the <b>Tit for Tat</b> bot which repeats your action, you could gain a maximum of 10*2 = 20 points. But if you always defected, you would win the first one, but the bot would defect afterwards, totalling points to 3 + 1*9 = 12. In the long run, choosing a greedy option might not be the right choice, as others know what you have done in the past and shape their actions around that.</p>
                      <br/>
                      <p>Next we'll see a simulation of these bots playing against each other.`,
        },
        { component: Simulation },
        {
            content: `<p>The Always Cooperate bot scored a total of 600 points.</p>
                      <p>The Always Defect bot scored a total of 702 points.</p>
                      <p>The Tit for Tat bot scored a total of 699 points.</p>
                      <p>The Pavlov bot scored a total of 650 points.</p>
                      <br/>
                      <p>Even though Always Defect scored the most in this case, it was close between that and Tit for Tat. Furthermore, in many studies of this game, the game is evolutionary (winners repopulate) and schocastic (small chance of randomness). In these cases which are more reflective of the real world, TFT and other strategies that are able to cooperate tend to perform better and win over time.</p>`,
        },
        {
            content: `<p>Thanks for playing!</p>`,
        },
    ];

    let currentSlide = 0;

    // Reactively update the disableMove based on currentSlide and gamesPlayedOnCurrentSlide
    $: disableForward =
        (slides[currentSlide].component &&
            slides[currentSlide].component == Game &&
            gamesPlayedOnCurrentSlide < NUM_GAMES) ||
        (slides[currentSlide].component &&
            slides[currentSlide].component == Simulation &&
            !simDone);

    $: disableBackward = currentSlide > 5;

    const nextSlide = () => {
        if (
            slides[currentSlide].component &&
            slides[currentSlide].component == Game
        ) {
            gamesPlayedOnCurrentSlide = 0; // Reset counter when moving away from slides 6 or 7
        }
        currentSlide = Math.min(currentSlide + 1, slides.length - 1);
    };

    const prevSlide = () => {
        if (
            slides[currentSlide].component &&
            slides[currentSlide].component == Game
        ) {
            gamesPlayedOnCurrentSlide = 0; // Reset counter when moving away from slides 6 or 7
        }
        currentSlide = Math.max(0, currentSlide - 1);
    };

    onMount(() => {
        const handleKeyDown = (event) => {
            if (!disableBackward) {
                if (event.key === "ArrowLeft") {
                    prevSlide();
                }
            }

            if (!disableForward) {
                if (event.key === "ArrowRight") {
                    nextSlide();
                }
            }
        };

        document.addEventListener("keydown", handleKeyDown);

        return () => {
            document.removeEventListener("keydown", handleKeyDown);
        };
    });
</script>

<div>
    <p style="font-size: small">Slide {currentSlide + 1}</p>
    {#if slides[currentSlide].content}
        {@html slides[currentSlide].content}
    {:else if slides[currentSlide].component}
        <svelte:component
            this={slides[currentSlide].component}
            {...slides[currentSlide].props}
            on:gamePlayed={handleGamePlayed}
            on:simDone={() => {
                simDone = true;
            }}
        />
    {/if}

    <button on:click={prevSlide} disabled={disableBackward}>Previous</button>
    <button on:click={nextSlide} disabled={disableForward}>Next</button>

    <div class="slides-navigation">
        {#each slides as _, index}
            <!-- svelte-ignore a11y-click-events-have-key-events -->
            <!-- svelte-ignore a11y-no-noninteractive-tabindex -->
            <!-- svelte-ignore a11y-no-static-element-interactions -->
            <span
                class="dot"
                class:active={index === currentSlide}
                class:past={index < currentSlide}
                on:click={() => {
                    currentSlide = index;
                }}
                tabindex="0"
                aria-label={`Go to slide ${index + 1}`}
            ></span>
        {/each}
    </div>
</div>

<style>
    /* Add your CSS styles here */
    button {
        background-color: #9ba0a8; /* Blue background */
        color: #ffffff; /* White text */
        padding: 10px 20px; /* Top and bottom padding, left and right padding */
        border: none; /* No border */
        border-radius: 5px; /* Rounded corners */
        cursor: pointer; /* Cursor changes to a pointer on hover */
        font-size: 16px; /* Larger font size */
        transition: background-color 0.6s; /* Smooth transition for background color */
    }

    button:hover {
        background-color: #0056b3; /* Darker blue on hover */
    }

    button:disabled {
        background-color: #cccccc; /* Gray background for disabled state */
        cursor: not-allowed; /* Cursor indicates button is not clickable */
    }
    .slides-navigation {
        position: absolute;
        top: 10px;
        left: 10px;
        z-index: 10;
    }

    .dot {
        height: 10px;
        width: 10px;
        margin: 0 5px;
        background-color: #bbb;
        border-radius: 50%;
        display: inline-block;
        transition: background-color 0.6s ease;
        cursor: pointer;
    }

    .dot:hover {
        background-color: #717171;
    }

    .dot.active {
        transform: scale(1.5);
    }

    .dot.past {
        background-color: lightgreen; /* Light green color for visited slides */
    }
</style>
