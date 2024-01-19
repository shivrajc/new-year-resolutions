<script>
    import { data } from "../data/data";
    import {scaleLinear, scaleBand, range} from 'd3';
    import { slide } from 'svelte/transition';
	import { linear } from 'svelte/easing';
    import ButtonGroup from "./ButtonGroup.svelte";
    import Header from "./Header.svelte";
    import Footer from "./Footer.svelte";
    
    export let width;
    export let height;    

    let hoveredData;    
    let viewGender = false;
    let sortBy = "all";
    let sortOrder = "desc";

    const focusList = ["Doing more exercise or improving fitness", "Saving more money", "Losing weight", "Improving my diet"]
    const genderColumns = ["all", "male", "female"];
    const ageGroupColumns = ["18 to 24", "25 to 49", "50 to 64", "65 plus"];
    const columns = [...new Set($data.map(d => d.criteria))];
    const paddingInner = 0.2;
    
    $: selectedColumns = viewGender ? genderColumns : ageGroupColumns;
    $: resolutions = [...new Set($data.map(d => d.resolution))];

    $: margin = {top: 40, right: 60, bottom: 40, left: 40};
    $: boundedWidth = width - margin.left - margin.right;
    $: boundedHeight = height - margin.top - margin.bottom;

    $: yScale = scaleBand()
        .domain(range(1, resolutions.length+1))
        .range([0, boundedHeight]);
        
    $: xScaleWidth = scaleBand()
        .domain(selectedColumns)
        .range([0, boundedWidth]).paddingInner(paddingInner);

    $: xScale = scaleLinear()
        .domain([0, 100])
        .range([0, xScaleWidth.bandwidth()]);

    function handleHover(resolution) {
        hoveredData = resolution;
    }        
        
    function y2(column, d) {        
        return $data.filter(c => c.criteria === selectedColumns[selectedColumns.indexOf(column)+1]).filter(a => a.resolution === d.resolution)[0].rank;
    }
</script>


<main>
    <Header />
    <ButtonGroup {viewGender} on:button-click={(e) => viewGender = e.detail.viewGender}/>
    <svg {width} {height} transition:slide={{ delay: 250, duration: 1000, easing: linear, axis: 'y' }}>    
        <g transform="translate({margin.left}, {margin.top})">
            {#each selectedColumns as column, i}                
                <text 
                    x={xScaleWidth(column)} 
                    y={8}
                    class="data-header"
                >
                    {column}
                </text>
                {#each $data.filter(c => c.criteria === column).sort((a, b) => (sortOrder === "asc") && (sortBy === column) ? a.value - b.value : b.value - a.value) as d}                                                        
                    {#if i<selectedColumns.length-1}
                        <line x1={xScaleWidth.step() * (i +1) - (xScaleWidth.step() * paddingInner) + 16} 
                            x2={xScaleWidth.step() * (i+1) - 4}
                            y1={yScale(d.rank) + (yScale.bandwidth()/2) + 36}                                                        
                            y2={yScale(y2(column, d)) + (yScale.bandwidth()/2) + 36}
                            stroke-width="1.6"
                            stroke-dasharray="2"
                            class:hovered-line={hoveredData == d.resolution}
                        />            
                    {/if}

                    <rect
                        x={xScaleWidth(column)} 
                        y={yScale(d.rank) + yScale.bandwidth() / 2 + 3.5}
                        width={xScaleWidth.bandwidth()}
                        height={yScale.bandwidth()}
                        class="transparent-rect"                                                                        
                        on:mouseover={() => handleHover(d.resolution)}    
                        on:mouseout={() => hoveredData = null}
                    >
                    </rect>
                    <g transform="translate(0,32)">
                    <rect
                        x={xScaleWidth(column)} 
                        y={yScale(d.rank) + yScale.bandwidth() / 2 + 3.5}
                        width={String(d.value).length === 1 ? xScaleWidth.bandwidth()-16 : xScaleWidth.bandwidth()-22}
                        height={1}
                        class="rect-base"
                        rx=0
                        class:focused={focusList.includes(d.resolution)}
                        class:hovered={hoveredData == d.resolution}
                    >
                    </rect>
                    <rect
                        x={xScaleWidth(column)} 
                        y={yScale(d.rank) + yScale.bandwidth() / 2}
                        width={xScale(d.value)}
                        height={8}
                        class="rect-top"
                        rx=4
                        class:focused={focusList.includes(d.resolution)}
                        class:hovered={hoveredData == d.resolution}
                    ></rect>
                    <text 
                        x={xScaleWidth(column)} 
                        y={yScale(d.rank) + yScale.bandwidth() / 3}
                        class="main-labels"
                        class:focused={focusList.includes(d.resolution)}
                        class:hovered={hoveredData == d.resolution}
                    >
                        {d.resolution}
                    </text>

                    <text 
                        x={String(d.value).length === 1 ? xScaleWidth(column) + xScaleWidth.bandwidth() - 13 : xScaleWidth(column) + xScaleWidth.bandwidth() - 20}
                        y={yScale(d.rank) + yScale.bandwidth() / 2 + 4}
                        class="value-label"
                        class:focused={focusList.includes(d.resolution)}
                        class:hovered={hoveredData == d.resolution}
                    >
                        {String(d.value) + "%"}                        
                    </text>
                </g>
                {/each}        
            {/each}            
        </g>
    </svg>
    <Footer />
</main>

<style>
    main {        
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        padding: 20px 20px 20px 20px;
        margin: 0 auto;
        text-align: center;
        font-family: "Open Sans", sans-serif;
    }


    .transparent-rect {
        fill: transparent;
        cursor: pointer;
    }
    .rect-base {
        fill: var(--primary-light-gray);
        stroke: none;
        stroke-width: 1;
        pointer-events: none;
    }

    .rect-top {
        fill: var(--primary-gray);
        stroke: none;
        pointer-events: none;
    }

    text {
        font-size: 16px;
        pointer-events: none;
    }

    .data-header {
        font-size: 18px;
        font-weight: 700;
        color: var(--dark-gray);
    }

    .main-labels {
        fill: var(--label-gray);
        font-family: "Alegreya Sans", serif;
        font-size: 18px;
    }

    .value-label-rect {
        fill: white;
    }
    .value-label {
        text-anchor: start;
        text-align: right;
        dominant-baseline: middle;        
        fill: var(--data-label-gray);
    }
    
    .focused {
        fill: var(--focus-color);
    }
    
    .hovered {
        fill: var(--highlight-color);
        cursor: pointer;
    }

    .hovered-line {
        stroke: var(--highlight-color);             
    }    
</style>