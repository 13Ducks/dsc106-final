<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";
    import { createEventDispatcher } from "svelte";

    const dispatch = createEventDispatcher();

    let strats = ["Cooperate", "Defect", "TFT", "Pavlov"];
    let fills = {
        Cooperate: "#7e549e",
        Defect: "#c2549d",
        TFT: "#fc8370",
        Pavlov: "#fecb3e",
    };

    let int;
    let timeout = 1000;

    export let key;

    let data = [0, 0];
    let svg;

    let lastComputerStrategy = "";
    let lastUserStrategy = "";
    let playCount = 0;

    const payoffs = {
        "Cooperate-Cooperate": { user: 2, opponent: 2 },
        "Cooperate-Defect": { user: 0, opponent: 3 },
        "Defect-Cooperate": { user: 3, opponent: 0 },
        "Defect-Defect": { user: 1, opponent: 1 },
    };

    const getChoice = (
        strategy,
        lastUserStrategy,
        lastComputerStrategy,
        playCount,
    ) => {
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

    onMount(() => {
        const margin = { top: 10, right: 30, bottom: 30, left: 40 };
        const width = 400 - margin.left - margin.right;
        const height = 170 - margin.top - margin.bottom;

        const x = d3
            .scaleBand()
            .domain(data.map((d, i) => i))
            .range([margin.left, width - margin.right])
            .padding(0.1);

        const y = d3
            .scaleLinear()
            .domain([0, d3.max(data)])
            .nice()
            .range([height - margin.bottom, margin.top]);

        const xAxis = (g) =>
            g
                .attr(
                    "transform",
                    `translate(0,${height - margin.bottom + 10})`,
                )
                .call(d3.axisBottom(x).tickFormat((i) => data[i]));

        const yAxis = (g) =>
            g
                .attr("transform", `translate(${margin.left},10)`)
                .call(d3.axisLeft(y));

        svg = d3
            .select("#chart" + key)
            .append("svg")
            .attr("width", width)
            .attr("height", height);

        const updateBars = () => {
            const strat1 = strats[Math.floor(key / 4)];
            const strat2 = strats[key % 4];

            const p1Choice = getChoice(
                strat1,
                lastUserStrategy,
                lastComputerStrategy,
                playCount,
            );
            const p2Choice = getChoice(
                strat2,
                lastComputerStrategy,
                lastUserStrategy,
                playCount,
            );
            playCount += 1;
            lastUserStrategy = p2Choice;
            lastComputerStrategy = p1Choice;
            const result = payoffs[`${p1Choice}-${p2Choice}`];
            data = [data[0] + result.user, data[1] + result.opponent];

            y.domain([0, d3.max(data)]);

            const bars = svg.selectAll("rect").data(data);
            const colors = [fills[strat1], fills[strat2]];

            bars.enter()
                .append("rect")
                .attr("fill", (d, i) => colors[i])
                .attr("x", (d, i) => x(i))
                .attr("y", y(0) + 10)
                .attr("height", 0)
                .attr("width", x.bandwidth())
                .merge(bars)
                .transition()
                .duration(1000)
                .attr("x", (d, i) => x(i))
                .attr("y", (d) => y(d) + 10)
                .attr("height", (d) => y(0) - y(d));

            svg.select(".x-axis").transition().duration(timeout).call(xAxis); // Update x-axis with transition
            svg.select(".y-axis").transition().duration(timeout).call(yAxis); // Update y-axis with transition

            if (key == 0) {
                dispatch("simPlayed", { playCount });
            }

            if (playCount % 10 == 0) {
                clearInterval(int);

                if (playCount < 100) {
                    timeout /= 1.5;
                    int = setInterval(updateBars, timeout);
                }
            }
        };

        int = setInterval(updateBars, timeout);

        svg.append("g")
            .attr("class", "x-axis") // Add class to x-axis group
            .call(xAxis);

        svg.append("g")
            .attr("class", "y-axis") // Add class to y-axis group
            .call(yAxis);

        svg.append("text")
            .attr("x", width / 2)
            .attr("y", 15)
            .attr("text-anchor", "middle")
            .style("font-size", "16px")
            .text(`${strats[Math.floor(key / 4)]} vs ${strats[key % 4]}`);
    });
</script>

<div id={"chart" + key}></div>
