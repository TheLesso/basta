<script lang="ts">
    import { onMount } from "svelte";
    import { page } from "$app/stores";
    import { fade } from "svelte/transition";
    import { base } from "$app/paths";

    let time = new Date();

    let config = {
        split: true,
        numbers: true,
        show_inactive_numbers: false,
        winter: 0,
        santa: false,
        true_binary_seconds: false,
        info: "2",
    };

    const to_bin = (num: number, pad: number = 6) =>
        num.toString(2).padStart(pad, "0");

    const weekdays = ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"];

    $: weekday = weekdays[time.getDay()];
    $: hours = to_bin(time.getHours(), 5).padStart(6, "-");
    $: minutes = to_bin(time.getMinutes());
    // get 64:th of a second seconds version: Math.round((time.getSeconds() * time.getMilliseconds()) / 937.5)
    $: seconds = to_bin(
        config.true_binary_seconds
            ? Math.floor(
                  (time.getSeconds() + time.getMilliseconds() / 1000) / 0.9375,
              )
            : time.getSeconds(),
    );

    onMount(() => {
        config = {
            split: !$page.url.searchParams.has("split"),
            numbers: !$page.url.searchParams.has("numbers"),
            show_inactive_numbers: $page.url.searchParams.has(
                "show_inactive_numbers",
            ),
            winter: Number.parseInt(
                $page.url.searchParams.get("winter") ?? "0",
            ),
            santa: $page.url.searchParams.has("santa"),
            true_binary_seconds: $page.url.searchParams.has(
                "true_binary_seconds",
            ),
            info: $page.url.searchParams.get("info") ?? "",
        };

        let handle: string | number | NodeJS.Timer | undefined;
        if (config.true_binary_seconds) {
            handle = setInterval(() => (time = new Date()), 937.5);
        } else {
            handle = setInterval(() => (time = new Date()), 1000);
        }

        return () => clearInterval(handle);
    });

    let odd_heart: boolean;
    $: odd_heart = hours.at(-1) === "1";

    const random_snowflake = () => `&#1005${Math.floor(Math.random() * 3 + 2)}`;

    /**
     * Returns amount of degrees rotated since midnight for h, m, s
     * @param now current time for rotations
     */
    const analog_rotate = (now: Date) => {
        const midnight = new Date(
            now.getFullYear(),
            now.getMonth(),
            now.getDate(),
            0,
            0,
            0,
        );
        const mil_delta = now.getTime() - midnight.getTime();
        // time since midnight
        const h = mil_delta / (1000 * 60 * 60);
        const m = h * 60;
        const s = m * 60;
        // convert time to rotations since midnight
        return {
            h: Math.round(h * 30 + h / 2),
            m: Math.round(m * 6),
            s: Math.round(s * 6),
        };
    };
</script>

{#if config.winter > 0}
    {#each Array(config.winter) as i}
        <div class="snow">{@html random_snowflake()}</div>
    {/each}
{/if}

<div class="container">
    {#each [hours, minutes, seconds] as row, i}
        <div class="row">
            {#each row.split("") as unit, j}
                {@const secondary =
                    config.split && !(i % 2 == 0 ? j % 2 == 0 : j % 2 == 1)}
                {#if unit === "-"}
                    <div
                        class="circle show_inactive_numbers"
                        style:visibility={"hidden"}
                        class:clock={config.info === "2"}
                        class:active={odd_heart}
                        class:secondary
                    ></div>
                {:else}
                    {@const active = Boolean(Number.parseInt(unit))}

                    <svg class="circle" class:active class:secondary xmlns="http://www.w3.org/2000/svg" width="30px" height="30px" viewBox="0 0 72 72" id="emoji">
                        <g id="color">
                          <path class="Badger_Color" fill="#2e2e2e" d="M66.345,45.6765c2.0234.0968,1.1253-25.4583-15.5132-25.51C45.4029,20.15,34.7472,22.99,29.32,23.1375c-1.3974.038-4.4352.048-5.5315-.8195-.6186-.49-.2131-2.0038-1.8438-2.5609-1.1324-.3868-2.5413.9451-3.38,1.2805a23.09,23.09,0,0,0-8.2973,4.9681c-1.0957,1.0031-4.95,2.0035-4.8145,3.4828.1849,2.0187,7.2729,3.5853,7.2729,3.5853s7.0964,1.9422,9.1168,3.2779c3.1615,2.09,10.39,10.7109,10.1412,11.268-.1254.28-4.8936.3328-4.8145,2.8682.0543,1.7409,2.7658,1.7414,2.7658,1.7414h5.0193c.9442-.6272,1.3347-3.1729,1.4305-4.3023.0854-1.0068-1.5759-3.6953-.611-3.995,2.4779-.77,4.77.3993,10.6533.6146.8446.0309,0,3.38,0,3.38s-4.6731,1.5124-4.61,2.8682c.0672,1.4341,2.356,1.4341,2.356,1.4341h5.634s4.0166-7.5875,4.712-8.1949c2.6539-2.3175,4.9935-1.83,9.1168-4.5072C63.6362,39.5272,64.2135,45.5746,66.345,45.6765Z"/>
                          <path  fill="#ffffff" d="M12.5971,27.7718c2.95-2.6329,12.8133-4.2885,12.0353-5.3581-2.488-3.4211-5.2288-1.764-6.0679-1.4286-2.2451.8973-5.8749,1.2566-8.2973,4.9682-.8119,1.244-5.8456,2.4133-4.8145,3.4828C7.922,31.9972,14.1355,32.5718,19.91,32.9a60.57,60.57,0,0,1,13.016,2.354S28.45,22.588,13.4142,29.8212C12.2059,30.4025,11.7034,28.5694,12.5971,27.7718Z"/>
                          <path class="Badger_Color" fill="##2e2e2e" d="M53.1224,51.7357l7.5048.4936s2.7368-3.1236,2.4876-5.7715c-.13-1.384-2.4173-5.7715-2.4173-5.7715l-4.49,1.5637,2.3011,4.3785-4.0365,1.9379Z"/>
                          <path class="Badger_Color" fill="##2e2e2e" d="M24.8284,51.2053c-1.0614,1.5379-7.4582-.4936-7.4582-.4936l1.0783-3.0751,4.6564-1.4519.6847-9.0428s3.2307,2.6146,3.5553,3.8918C27.99,43.5726,26.5268,48.7446,24.8284,51.2053Z"/>
                        </g>    
                        <!-- <text x="44" y="32" fill="black" text-anchor="middle" alignment-baseline="middle">123</text> -->
                        <g id="line">
                          <path id="Badger_Line_1" fill="none" stroke="#000000" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M66.6878,44.68c.9551-4.1388.7825-24.4613-15.856-24.5126C45.4029,20.15,34.7472,22.99,29.32,23.1375c-1.3974.038-4.4352.048-5.5315-.8195-.6186-.49-.2131-2.0038-1.8438-2.5609-1.1324-.3868-2.5413.9451-3.38,1.2805a23.3153,23.3153,0,0,0-8.2973,4.9681S5.4492,28.09,5.4527,29.4885c.0065,2.5891,7.2729,3.5853,7.2729,3.5853s7.0964,1.9422,9.1168,3.2779c3.1615,2.09,10.39,10.7109,10.1412,11.268-.1254.28-4.8936.3328-4.8145,2.8682.0543,1.7409,2.7658,1.7414,2.7658,1.7414h5.0193c.9442-.6272,1.3347-3.1729,1.4305-4.3023.0854-1.0068-1.5759-3.6953-.611-3.995,2.4779-.77,4.77.3993,10.6533.6146.8446.0309,0,3.38,0,3.38s-4.6731,1.5124-4.61,2.8682c.0672,1.4341,2.356,1.4341,2.356,1.4341h5.634s4.0166-7.5875,4.712-8.1949c2.6539-2.3175,4.9935-1.83,9.1168-4.5072"/>
                          <path id="Badger_Line_2" fill="none" stroke="#000000" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M55.8961,43.0439l3.2286,3.9687c-.6286.92-7.115.4826-6.0023,4.7231.1348.5136,1.5354.49,2.0664.492,1.0578.0036,4.2311,0,4.2311,0"/>
                          <path id="Badger_Line_3" fill="none" stroke="#000000" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M22.55,36.88s2.1119,7.22.8225,9.1084c-.6286.92-7.115.4826-6.0023,4.7231.1348.5136,1.5354.49,2.0664.492,1.0578.0036,4.2311,0,4.2311,0"/>
                          <path id="Badger_Line_4" d="M7.9183,27.2494l.2742,4.7876c-.9434.379-3.314-2.3569-3.314-2.3569S7.03,26.853,7.9183,27.2494Z" fill="#000000"/>
                        </g>
                
                      </svg>
                {/if}
            {/each}
        </div>
    {/each}
</div>

{#if minutes.at(-1) === "1"}
    <p class="footer" transition:fade>
        Made with <svg viewBox="0 0 1792 1792" preserveAspectRatio="xMidYMid meet" xmlns="http://www.w3.org/2000/svg"><path class="heart" class:odd_heart d="M896 1664q-26 0-44-18l-624-602q-10-8-27.5-26T145 952.5 77 855 23.5 734 0 596q0-220 127-344t351-124q62 0 126.5 21.5t120 58T820 276t76 68q36-36 76-68t95.5-68.5 120-58T1314 128q224 0 351 124t127 344q0 221-229 450l-623 600q-18 18-44 18z"></path></svg>  by RootM</p>
{:else}
    <p class="footer" transition:fade>
        Fork me on <span class="url" class:odd_heart
            >github.com/stagrim/basta</span
        >
    </p>
{/if}

<style lang="scss">
    //TODO: break out some styling parts to different files
    @font-face {
        font-family: 'Roboto Mono';
        src: url('/RobotoMono-VariableFont_wght.ttf');
    }

    $background: #0e0e0e;
    $inactive: #2e2e2e;
    $main: #f280a1;
    $secondary: #9966cc;
    $pinkBadger: "";

    $diameter: 16vw;
    $diameter-height-breakpoint: 26vh;

    $diameter-vertical: 15vh;
    $diameter-height-breakpoint-vertical: 26vw;

    :global(body) {
        background-color: $background;
        font-size: 16px;
    }

    * {
        font-family: "Roboto", "Helvetica", "Arial", sans-serif;
    }

    .container {
        display: flex;
        flex-direction: column;
        align-items: center;
        margin-top: 2%;
    }

    .row {
        display: flex;
    }

    .circle{
        width: min($diameter, $diameter-height-breakpoint);
        height: $diameter;
        max-height: $diameter-height-breakpoint;
        margin: 5px;
        //background-color: $inactive;
        fill: $inactive;
        
        border-radius: 50%;
        transition: 0.1s;

        display: flex;
        justify-content: center;
        vertical-align: middle;
        align-items: center;
        color: $inactive;
        
    }

    .Badger_Color {
        fill: $inactive;
    }



    .circle.secondary.show_inactive_numbers {
        color: $secondary;
    }
    


    .circle.active .Badger_Color {
        //background-color: $main;
        //color: $inactive;
        fill: $main;
    }

    .circle.secondary.active .Badger_Color {
        //background-color: $secondary;
        fill: $secondary;
    }

    .circle .legend {
        font-family: Roboto Mono;
        font-weight: 700;
        font-size: 3.5rem;
    }

    .footer {
        bottom: 0;
        width: 100%;
        position: fixed;
        text-align: center;
        color: #787878;
        font-size: 1.66rem;
    }

    .footer .url {
        color: $main;
        text-decoration: underline;
    }

    .footer .url.odd_heart {
        color: $secondary;
    }

    .heart {
        fill: $main;
        transition: 1s;
    }

    .footer svg {
        height: 1.8rem;
    }

    .heart.odd_heart {
        fill: $secondary;
    }

    @media (orientation: portrait) {
        .clock {
            box-shadow: 0 0 0 0.5vh $main inset;
        }

        .container {
            justify-content: center;
            flex-direction: row;
        }

        .row {
            display: flex;
            flex-direction: column-reverse;
        }

        .circle {
            height: min($diameter-vertical, $diameter-height-breakpoint-vertical);
            width: $diameter-vertical;
            max-width: $diameter-height-breakpoint-vertical;
        }

        .tomte {
            height: min($diameter, $diameter-height-breakpoint);
        }
    }

    .tomte {
        width: min($diameter, $diameter-height-breakpoint + 0.3vw);
        max-height: $diameter-height-breakpoint;
        height: $diameter;
        left: calc(($diameter / 2) - 3.2vw);
        top: 46vh;
        position: absolute;
    }

    // Snow effects

    :global(body) {
        /* height: 100vh; */
        /* background: radial-gradient(ellipse at bottom, #1b2735 0%, #090a0f 100%); */
        overflow: hidden;
    }

    @function random_range($min, $max) {
        $rand: random();
        $random_range: $min + floor($rand * (($max - $min) + 1));
        @return $random_range;
    }

    .snow {
        // filter: drop-shadow(0 0 10px white);
        // $total: 200;
        position: absolute;
        z-index: 10;
        width: 10px;
        height: 10px;
        // background: white;
        // border-radius: 50%;
        color: white;

        @for $i from 1 through 1000 {
            $random-x: random(1000000) * 0.0001vw;
            $random-offset: random_range(-100000, 100000) * 0.0001vw;
            $random-x-end: $random-x + $random-offset;
            $random-x-end-yoyo: calc($random-x + ($random-offset / 2));
            $random-yoyo-time: calc(random_range(30000, 80000) / 100000);
            $random-yoyo-y: $random-yoyo-time * 100vh;
            $random-scale: random(3);
            $fall-duration: random_range(10, 30) * 1s;
            $fall-delay: random(30) * -1s;
            $rotate: random(720) * 1deg - 360deg;

            &:nth-child(#{$i}) {
                opacity: random(7000) * 0.0001 + 0.3;
                transform: translate($random-x, -60px) scale($random-scale) rotate($rotate);
                animation: fall-#{$i} $fall-duration $fall-delay linear infinite;
            }

            @keyframes fall-#{$i} {
                #{percentage($random-yoyo-time)} {
                    transform: translate($random-x-end, $random-yoyo-y) scale($random-scale);
                }

                to {
                    transform: translate($random-x-end-yoyo, 100vh) scale($random-scale) rotate($rotate);
                }
            }
        }
    }

    // Analog clock

    .clock {
        box-shadow: 0 0 0 0.5vw $main inset;
        display: flex;
        justify-content: center;
        position: relative;
    }

    $blob-diameter: 10%;
    #blob {
        position: absolute;
        background: $main;
        width: $blob-diameter;
        height: $blob-diameter;
        border-radius: 50%;
    }

    #hour,
    #minute,
    #second {
        position: absolute;
        background-color: $main;
        transform-origin: bottom center;
        border-radius: 1px;
        transition: 0.99s linear;
    }

    .active {
        #hour,
        #minute,
        #second,
        #blob {
            background: $inactive;
        }
    }

    $hour-length: 20%;
    #hour {
        width: 5%;
        height: $hour-length;
        margin-top: -$hour-length;
    }

    $minute-length: 35%;
    #minute {
        width: 5%;
        height: $minute-length;
        margin-top: -$minute-length;
    }

    $second-length: 40%;
    #second {
        width: 2%;
        height: $second-length;
        margin-top: -$second-length;
    }
</style>
