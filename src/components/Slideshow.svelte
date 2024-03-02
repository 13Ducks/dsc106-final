<script>
    import { onMount } from "svelte";
    import Game from "./Game.svelte";

    const NUM_GAMES = 2;

    let currentScore = 0;
    let gamesPlayedOnCurrentSlide = 0;
    let disableForward = false;
    let disableBackward = false;

    const handleGamePlayed = (event) => {
        let data = event.detail;
        gamesPlayedOnCurrentSlide = data["playCount"];
        currentScore = data["userPayoff"];
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
            content: `<p>You'll play 10 times against 4 imaginary opponents with different playstyles. Let's see how you do! For simplicity, we'll change the terminology (testify = defect and silent = cooperate) and reverse the game so a bigger number is better (defecting while the other cooperates is +3)</p>`,
        },
    ];

    let currentSlide = 0;

    // Reactively update the disableMove based on currentSlide and gamesPlayedOnCurrentSlide
    $: disableForward =
        slides[currentSlide].component &&
        slides[currentSlide].component == Game &&
        gamesPlayedOnCurrentSlide < NUM_GAMES;

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
    <p style="font-size: small">Slide {currentSlide}</p>
    {#if slides[currentSlide].content}
        {@html slides[currentSlide].content}
    {:else if slides[currentSlide].component}
        <svelte:component
            this={slides[currentSlide].component}
            {...slides[currentSlide].props}
            on:gamePlayed={handleGamePlayed}
        />
    {/if}

    <button on:click={prevSlide} disabled={disableBackward}>Previous</button>
    <button on:click={nextSlide} disabled={disableForward}>Next</button>
</div>

<style>
    /* Add your CSS styles here */
</style>