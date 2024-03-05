<script>
    import { onMount, onDestroy, tick } from "svelte";
    import { fade } from "svelte/transition";
    import { createEventDispatcher } from "svelte";

    const dispatch = createEventDispatcher();
    let playCount = 0;
    let computerChoice = "";

    let lastUserStrategy = "";
    let lastComputerStrategy = "";
    let userStrategy = "";
    let userPayoff = 0;
    let opponentPayoff = 0;
    let history = [];
    let animating = false; // Animation trigger

    const strategies = ["Cooperate", "Defect"];
    const NUM_GAMES = 10;

    const payoffs = {
        "Cooperate-Cooperate": { user: 2, opponent: 2 },
        "Cooperate-Defect": { user: 0, opponent: 3 },
        "Defect-Cooperate": { user: 3, opponent: 0 },
        "Defect-Defect": { user: 1, opponent: 1 },
    };

    export let opponentStrategy = "Cooperate"; // Default opponent strategy
    // Update computerChoice whenever userStrategy changes
    $: if (userStrategy) {
        computerChoice = getChoice(opponentStrategy);
    }
    // Reactive statement to update CSS variable based on userStrategy
    $: {
        const rootStyle = document.documentElement.style;
        if (userStrategy === "Defect") {
            rootStyle.setProperty("--user-translateX-vw", "18vw");
        } else if (userStrategy === "Cooperate") {
            rootStyle.setProperty("--user-translateX-vw", "16vw");
        }
    }

    const getChoice = (strategy) => {
        if (strategy == "Cooperate") return "Cooperate";
        if (strategy == "Defect") return "Defect";
        if (strategy == "TFT") {
            if (playCount == 0) {
                return "Cooperate";
            } else {
                return lastUserStrategy;
            }
        }
        if (strategy == "Pavlov") {
            if (playCount == 0) {
                return "Cooperate";
            } else if (lastUserStrategy == lastComputerStrategy) {
                return "Cooperate";
            } else {
                return "Defect";
            }
        }
    };

    const playGame = async () => {
        if (!userStrategy) return; // Do nothing if no strategy is selected
        computerChoice = getChoice(opponentStrategy);

        showChoices = true;
        animating = true; // Start animation
        await tick(); // Ensure UI updates with the animation start
        setTimeout(() => {
            const computerChoice = getChoice(opponentStrategy);
            lastUserStrategy = userStrategy;
            lastComputerStrategy = computerChoice;
            const resultKey = `${userStrategy}-${computerChoice}`;
            userPayoff += payoffs[resultKey].user;
            opponentPayoff += payoffs[resultKey].opponent;
            const newRound = {
                round: history.length + 1,
                userPayoff,
                opponentPayoff,
                userStrategy,
                computerChoice,
            };
            history = [...history, newRound];
            playCount++;
            dispatch("gamePlayed", { playCount, userPayoff });

            animating = false; // Reset animation trigger after completion
            showChoices = false;
        }, 900); // Adjusted to account for the animation delay and duration
    };
</script>

<div>
    <div class="game-layout">
        <div class="game-play">
            <h3>Choose Your Strategy:</h3>
            {#each strategies as strategy}
                <label>
                    <input
                        type="radio"
                        bind:group={userStrategy}
                        value={strategy}
                    />{strategy}
                </label>
            {/each}
            <button
                on:click={playGame}
                disabled={animating ||
                    userStrategy === "" ||
                    playCount >= NUM_GAMES}>Play</button
            >

            <div class="choice-container">
                <div
                    class="choice user-choice {animating ? 'animate-user' : ''}"
                >
                    {#if userStrategy}{userStrategy}{/if}
                    <span class="score user-score">{userPayoff}</span>
                </div>
                <div
                    class="choice computer-choice {animating
                        ? 'animate-computer'
                        : ''}"
                >
                    {#if animating}{computerChoice}{/if}
                    <!-- Show computer choice when userStrategy is set -->
                    <span class="score opponent-score">{opponentPayoff}</span>
                </div>
            </div>
        </div>
        <div class="game-history">
            <h3>Results:</h3>
            <p>User Score: {userPayoff}</p>
            <p>Opponent Score: {opponentPayoff}</p>

            <h3>Game History:</h3>
            <table transition:fade>
                <tr
                    ><th>Round</th><th>Your Strategy</th><th
                        >Opponent Strategy</th
                    ><th>Your Score</th><th>Opponent Score</th></tr
                >
                {#each history as round, i (round.round)}
                    <tr transition:fade={{ delay: i * 100 }}>
                        <td>{round.round}</td>
                        <td>{round.userStrategy}</td>
                        <td>{round.computerChoice}</td>
                        <td>{round.userPayoff}</td>
                        <td>{round.opponentPayoff}</td>
                    </tr>
                {/each}
            </table>
        </div>
    </div>
</div>

<style>
    :root {
        --user-translateX: 400%; /* Default value, can be adjusted */
    }

    .choice-container {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 20px; /* Increased padding */
        overflow: visible; /* Ensure animations can extend beyond the container */
    }

    .choice {
        position: relative; /* Needed for absolute positioning of pseudo-elements */
        transition: transform 0.4s ease-in-out;
    }

    .user-choice,
    .computer-choice {
        transform: translateX(0%);
    }

    .animate-user,
    .animate-computer {
        /* Adjusted for collision */
        /* Assuming 930% and -625% are your desired collision points */
        transition:
            transform 0.4s ease-in-out,
            opacity 0.4s ease;
    }

    @keyframes moveToCollision {
        0% {
            transform: translateX(0%); /* Start from original position */
        }
        100% {
            /* Adjust these values to the point where they should collide */
            transform: translateX(
                calc(50% - 50px)
            ); /* Example for user choice */
        }
    }

    .animate-user {
        transform: translateX(var(--user-translateX-vw));
    }

    .animate-computer {
        /* Assuming -425% is the desired end point, convert this to an equivalent vw unit based on your layout */
        transform: translateX(
            -13vw
        ); /* Example value, adjust based on your layout */
    }

    /* Pseudo-elements for scores */
    .user-choice::after,
    .computer-choice::after {
        content: attr(
            data-score
        ); /* Use the data-score attribute to set the score */
        position: absolute;
        top: 0;
        left: 50%;
        transform: translateX(-50%) translateY(-100%); /* Start above the choice */
        opacity: 0;
        transition:
            transform 0.4s ease-in-out,
            opacity 0.4s ease;
    }

    /* Animate scores away upon collision */
    .animate-user::after {
        transform: translateX(-50%) translateY(-200%); /* Move score up and away */
        opacity: 1;
    }

    .animate-computer::after {
        transform: translateX(-50%) translateY(-200%); /* Move score up and away */
        opacity: 1;
    }
    @keyframes popScore {
        0% {
            transform: scale(0) translateY(0); /* Start from scale 0, no vertical movement */
            opacity: 0; /* Start fully transparent */
        }
        30% {
            transform: scale(1.2) translateY(-20px); /* Slightly overshoot the scale and position for impact */
            opacity: 1; /* Fully visible */
        }
        60% {
            transform: scale(1) translateY(-15px); /* Settle back to final scale and position */
            opacity: 1; /* Remain fully visible */
        }
        100% {
            transform: scale(1) translateY(-15px); /* Remain at final scale and position */
            opacity: 1; /* Remain fully visible */
        }
    }

    .score {
        display: inline-block;
        margin-left: 10px; /* Space the score from the strategy text */
        transition:
            transform 0.4s ease-in-out,
            opacity 0.4s ease; /* Match the choice animations */
    }

    .animate-user .user-score,
    .animate-computer .opponent-score {
        transform: translateX(0) translateY(-20px); /* Adjust to control the emergence point */
        opacity: 1; /* Make scores visible upon collision */
    }
    .game-layout {
        display: flex;
        justify-content: space-between; /* Aligns children (game play and history) on each side */
    }

    .game-play,
    .game-history {
        flex: 1; /* Each takes up equal width */
        margin: 10px; /* Optional: Adds some space around each section */
    }

    /* Additional styles to ensure the game play area and history are well-presented */
    .game-play {
        max-width: 50%; /* Limit the width to avoid stretching too much */
    }

    .game-history {
        max-width: 50%; /* Limit the width for a more readable table */
    }
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
    }
</style>
