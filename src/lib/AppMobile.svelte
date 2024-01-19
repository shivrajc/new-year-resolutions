<script>
    import { data } from "../data/data";
    import {scaleLinear, scaleBand, scaleOrdinal, line, curveCardinal} from 'd3';
    import { slide } from 'svelte/transition';
	import { linear } from 'svelte/easing';
    import ButtonGroup from "./ButtonGroup.svelte";
    import HeaderMobile from "./HeaderMobile.svelte";
    import Footer from "./Footer.svelte";
    import ColorLegend from "./ColorLegend.svelte";
    
    export let width;
    export let height;    

    let hoveredData = null;    
    let viewGender = false;
    let lineData = {};

    const focusList = ["Doing more exercise or improving fitness", "Saving more money", "Losing weight", "Improving my diet"]
    const genderColumns = ["all", "male", "female"];
    const ageGroupColumns = ["18 to 24", "25 to 49", "50 to 64", "65 plus"];
    
    $: selectedColumns = viewGender ? genderColumns : ageGroupColumns;
    $: selectedColors = viewGender ? ["#000000", "#20B2AA", "#FFA500"] : ["#FF1493", "#1E90FF", "#FF8C00", "#DC143C"];
    $: resolutions = [...new Set($data.map(d => d.resolution))];

    $: margin = {top: 40, right: 20, bottom: 20, left: 20};
    $: boundedWidth = width - margin.left - margin.right;
    $: boundedHeight = height - margin.top - margin.bottom;

    $: yScale = scaleBand()
        .domain(resolutions)
        .range([0, boundedHeight]);

    $: yScale2 = scaleBand()        
        .range([0, boundedHeight]);

    $: xScale = scaleLinear()
        .domain([0, 100])
        .range([0, boundedWidth]);

    $: colorScale = scaleOrdinal()
        .domain(selectedColumns)
        .range(selectedColors);

    $: lineGenerator = line().curve(curveCardinal);

    $: selectedColumns.forEach(column => {
        let columnData = $data.filter(d => d.criteria === column);
        columnData.sort((a, b) => resolutions.indexOf(a.resolution) - resolutions.indexOf(b.resolution))
        let arr = [];
        yScale2.domain([...new Set(columnData.map(d => d.resolution))]);
        columnData.forEach(r => {
            arr.push([xScale(r.value), yScale2(r.resolution) + yScale2.bandwidth() / 2 + 3.5])
        })
        lineData[column] = arr;
    })

    function handleLegendClick(e) {
        hoveredData === e.detail.column ? hoveredData = null : hoveredData = e.detail.column;
    }
</script>

<main>
    <HeaderMobile />
    <ButtonGroup {viewGender} on:button-click={(e) => viewGender = e.detail.viewGender}/>
    <ColorLegend {colorScale} {selectedColumns} on:legend-click={handleLegendClick} />         
    <svg {width} {height} transition:slide={{ delay: 250, duration: 1000, easing: linear, axis: 'y' }}>    
        <g transform="translate({margin.left}, {margin.top})">
            {#each selectedColumns as column, i}
                <path
                    d={lineGenerator(lineData[column])}
                    fill="none"
                    stroke={colorScale(column)}
                    opacity={hoveredData === column ? "1": "0"}
                    stroke-width="1.4"
                    stroke-dasharray="2"                    
                ></path>
            {/each}
            {#each resolutions as resolution}
                <rect
                    x={xScale(0)} 
                    y={yScale(resolution) + yScale.bandwidth() / 2 + 3.5}
                    width={boundedWidth}
                    height={1}
                    class="rect-base"
                    rx=0
                    class:focused={focusList.includes(resolution)}
                >                
                </rect>
                <text
                    x={xScale(0)} 
                    y={yScale(resolution) + 20}
                    class="main-labels"
                    class:focused={focusList.includes(resolution)}                    
                >                
                    {resolution}
                </text>                
                {#each $data.filter(d => d.resolution === resolution).filter(c => selectedColumns.includes(c.criteria)) as d}
                    <circle
                        cx={xScale(d.value)} 
                        cy={yScale(resolution) + yScale.bandwidth() / 2 + 3.5}
                        r={6}
                        class="dots-criteria"    
                        fill={colorScale(d.criteria)}                
                    >
                    </circle>
                    <text
                        x={xScale(d.value) + (String(d.value).length === 1 ? 16 : 20)} 
                        y={yScale(resolution) + yScale.bandwidth() / 2 + 12}                        
                        class="data-labels"    
                        fill={colorScale(d.criteria)}   
                        opacity={hoveredData === d.criteria ? "1" : "0"}             
                    >
                    {d.value}%
                    </text>
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
        padding: 40px 20px 20px 20px;
        margin: 0 auto;
        text-align: center;
        font-family: "Open Sans", sans-serif;
    }

    .rect-base {
        fill: var(--primary-gray);
    }
    .dots-criteria {
        opacity: 0.9;
    }

    .main-labels {
        font-family: "Alegreya Sans", serif;
        font-size: 20px;
    }

    .data-labels {
        text-anchor: middle;
        dominant-baseline: hanging;
        font-size: 14px;
        font-weight: 500;
    }
</style>