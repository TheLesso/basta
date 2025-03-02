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

                    <svg class="circle" class:active class:secondary
                        id="Layer_1"
                        data-name="Layer 1"
                        xmlns="http://www.w3.org/2000/svg"
                        viewBox="0 0 1000 1000"
                        ><defs
                            ><style>
                                .Badger_Color {
                                    fill: #231f20;
                                    fill-rule: evenodd;
                                }
                            </style></defs
                        ><path
                            class="Badger_Color"
                            d="M963.8563,346.4149c6.736-8.9659,5.6507-18.54,1.2178-21.6819-2.9575-2.0955-7.3659-1.325-10.627-.0263a46.2909,46.2909,0,0,0-10.1246-15.0734c22.2618-4.1,34.1424-21.0624,31.8995-31.2407-2.3046-10.4587-21.1645-21.2135-43.7835-14.2251q-.1062-4.2065-.2125-8.4127a79.1862,79.1862,0,0,0,16.8388-16.0839C980.2339,199.3661,967.74,131.8335,937.3212,89.07c-31.4748-44.2477-84.3344-60.4007-121.1655-64.51-54.3542-6.064-107.6478,12.1783-149.3585,46.5524A263.2571,263.2571,0,0,0,533.2843,43.3047C529.3525,31.949,519.6487,24.8083,509.5722,24.53a25.1,25.1,0,0,0-18.1571,7.5521,28.0922,28.0922,0,0,0-11.7072-10.77,28.8019,28.8019,0,0,0-22.8533-.7841,30.5859,30.5859,0,0,0-13.154-2.5358c-12.4516.4484-23.1441,8.8261-29.5422,19.7493A196.5367,196.5367,0,0,0,273.19,58.3513,147.73,147.73,0,0,0,215.93,18.624c-12.3519-4.7314-52.934-10.0249-94.12-3.9872-31.1978,4.573-53.196,12.7359-63.542,28.95a28.9487,28.9487,0,0,0-4.9865,15.1234C52.9981,72.2913,63.611,84.47,77.1044,92.0391q24.6509,51.5952,49.3017,103.1908c-5.2791,13.6939-9.9134,27.4618-13.9894,41.56-3.9135,13.5361-7.2214,27.0564-9.9992,40.8711-9.2648-1.155-18.3341-2.132-27.6521-2.9793-14.341-1.3037-22.5466-1.576-25.4829,2.8478-5.7219,8.6215,10.2235,28.9986,21.8225,41.5029-12.1244,21.2153-16.9927,40.2686-13.9989,57.2193.9872,5.5891,3.975,17.5942-1.89,22.7284-3.3508,2.9335-8.8446,1.6024-10.5507,4.4564-2.3493,3.93,4.1159,12.3845,12.2939,20.9a55.8944,55.8944,0,0,0-13.8261-1.4894c-8.1026.1512-13.9773,2.8063-15.5877,4.4731a15.2934,15.2934,0,0,0-4.0164,11.313,16.94,16.94,0,0,0,9.1375,13.8483q29.3487,23.5764,58.6973,47.1534a32.9758,32.9758,0,0,0-9.1128,11.8824c-7.695,17.4351,3.399,37.2862,14.6235,56.3878,8.3585,14.2245,17.6437,29.9643,25.0809,42.4131,15.4312,52.09,29.9722,103.3427,44.1947,155.7752,13.2591,48.88,25.7563,97,37.9582,146.1542a180.1519,180.1519,0,0,0,219.4972,60.5616l43.0185,5.7532C645.7551,878.0038,780.4924,746.7205,859.9759,583.5577c4.0635-8.3415,7.8827-16.5492,11.6489-25.0324,18.5961-.9168,33.4079-12.6649,37.6849-27.8211,3.0467-10.7959.4994-22.1944-5.5717-31.3623q8.802-3.8058,17.6044-7.6117c6.6212-3.33,12.3191-8.5,12.7886-14.8423.3392-4.5819-2.1971-7.8582-6.3721-13.5507-2.4671-3.3638-4.862-6.7516-7.2828-10.2928,3.5346-1.0194,8.6878-2.5088,12.7032-3.6651,7.5234-2.1669,12.3282-3.6644,14.286-7.9695,1.15-2.53.3205-8.6727-3.8026-16.63a80.3561,80.3561,0,0,0-7.657-12.0111c13.0779-2.5006,24.996-9.4392,32.4262-20.557,5.9448-8.8949,10.3512-20.4979,6.7147-31.1574A29.9912,29.9912,0,0,0,963.8563,346.4149ZM837.52,87.7865c34.0245,15.8952,48.0892,66.1138,43.2685,71.2146-3.6657,3.8786-21.6376-24.7552-50.2084-40.58-27.1705-15.049-58.45-14.4022-58.1485-20.6571C772.7922,90.2833,808.0342,74.0116,837.52,87.7865ZM662.5317,541.5922c-.93,23.6408-42.4509,44.1786-66.0523,31.4766a34.0859,34.0859,0,0,1-16.9732-24.201c-2.1377-12.9514,4.5515-24.3854,10.0328-30.8,5.3984-6.3179,13.9144-12.6464,24.4622-14.1439C638.0132,500.5149,663.2764,522.6537,662.5317,541.5922ZM156.8923,587.2354c-5.3824,2.8592-32.1037-37.1925-49.1078-63.4589,7.8047-11.5065,11.25-22.4214,8.4928-32.0844-5.3-18.5786-35.4028-24.3219-35.0787-37.8971.0461-1.9315,1.6237-5.9848,5.5565-10.0282,4.6479-4.7791,12.7061-5.305,13.299-9.194,1.298-8.5184-18.9492-15.8294-17.4669-24.8138.8257-5.003,7.6787-5.62,8.4411-10.836,1.0605-7.2547-11.3735-11.7661-10.8136-19.063.6986-9.1032,20.4229-12.2991,20.6934-20.7023.2039-6.3395-11.5979-10.4145-10.6774-15.1331,1.0923-5.6,18.5318-4.208,20.7268-10.6313,3.3792-9.8874-31.3719-27.6258-28.74-34.4177,1.4823-3.8252,14.199-2.57,26.35-.517a372.3428,372.3428,0,0,1,29.5129,63.5852,372.493,372.493,0,0,1,18.7578,80.3242c-8.6129,22.8064-13.06,46.2491-12.604,70.234C144.9252,548.8857,161.38,584.8516,156.8923,587.2354ZM154.48,85.2736c-12.8672.43-28.0524.7348-32.2125-7.3857a11.6887,11.6887,0,0,1,1.3108-12.1637c12.1626-14.5818,70.6243,10.2173,69.1641,17.5189C192.2462,85.7241,178.1344,84.4818,154.48,85.2736ZM174.0435,526.72c2.1224-19.6951,20.185-33.901,40.3439-31.7291s34.78,19.8985,32.6577,39.5939-20.1849,33.9011-40.3439,31.7292S171.9212,546.4153,174.0435,526.72ZM292.8,909.4982c-.4424,1.3-2.4928,3.6463-6.1856,5.4859-6.5663,3.2712-12.8615,4.4151-18.1743,2.5166-8.9133-3.185-9.3419-12.72-17.0755-18.06-6.3681-4.397-15.7139-3.0714-20.7346-10.0642-2.2354-3.1139-4.095-8.26-2.0669-11.9684,1.2-2.1937,4.8225-4.37,11.7164-4.89,21.2152-1.599,39.08,14.9177,46.2082,22.8757C291.8162,901.3433,294.0488,905.83,292.8,909.4982Zm152.1484-20.5235c-3.0748,4.09-3.7567,7.485-6.8412,11.568-6.6092,8.7482-13.1636,9.697-20.3954,17.9378-3.8158,4.3484-6.0746,8.23-11.7592,10.4788-5.367,2.1236-10.2153,1.4464-14.6175.6624-7.7544-1.3808-16.3665-3.3962-20.1647-4.9689-9.28-3.8429-28.2674-13.7473-26.7325-21.8964.81-4.2991,7.16-7.017,13.7926-8.5818,11.3581-2.68,19.5755,2.1884,27.7714,1.8734,10.8412-.4174,29.567-9.6242,50.3454-25.8977,1.3139-1.0293,5.1678-4.0935,8.7126-2.9767,3.4932,1.1006,5.6377,5.9635,5.6237,10.1142C450.6671,882.1684,447.66,885.3682,444.9486,888.9747ZM959.1754,379.4692c-1.4548,4.7655-18.1178,6.1241-23.2538,8.1217-7.0907,2.7583-14.2017,9.3522-19.8592,12.9621-2.6439,1.687-6.7374,4.2561-8.2619,9.1776-1.7453,5.6336.6841,11.9247,4.8488,17.3015-18.406,1.7294-30.0336,16.6055-28.3427,29.3865,1.1285,8.5291,8.21,16.1814,18.003,19.8377a127.881,127.881,0,0,0-24.76,16.9086,21.4246,21.4246,0,0,0-1.2511,11.6106c1.3123,7.4049,6.5879,13.3447,13.3234,16.5368.6064,2.6048.5007,4.935-.715,6.6344-2.2911,3.203-7.6435,2.915-9.2659,2.9171-21.8933.03-50.669,53.9149-56.7877,64.9146-.0086.0154-33.0108,37.7042-49.7342,61.7594-19.6017,28.1954-60.4159,72.88-99.3831,112.3418-63.5519,64.36-136.27,120.6009-217.38,168.984,17.8447-30.1916,24.2722-65.826,12.8178-98.1762-4.1325-11.6718-21.4095-37.9864-50.1373-59.0954a190.3441,190.3441,0,0,0-47.076-25.2067Q361.76,736.8291,351.559,717.2728c-6.9165,7.2669-13.4453,14.692-19.766,22.4805-7.1959,8.8667-13.9319,17.9761-20.3022,27.4561a150.3988,150.3988,0,0,0-95.62,61.85q-7.9329-30.2184-15.8657-60.4369,32.5707-14.109,65.1417-28.2185A433.425,433.425,0,0,0,305.5938,583.12a433.4277,433.4277,0,0,0-21-163.344c1.2423-12.4652,3.2162-29.42,3.9921-37.0439,1.0382-10.1983,1.461-16.6875,5.94-23.97,2.9389-4.7785,5.8136-7.2133,7.5133-12.6138,1.857-5.9008-.1042-8.984,1.6086-14.9282,2.6848-9.3162,9.2136-9.5862,11.4872-18.3383,2.199-8.4647-3.63-13.15-.5009-21.3078,2.1848-5.6971,6.9748-6.49,8.679-13.0476,1.2365-4.7588-1.1728-6.5082.0854-11.1774,2.4356-9.037,13.4085-9.7278,15.9115-17.8616,1.7-5.5232-1.08-12.0907-6.8826-19.1986,11.2818-5.4974,18.5233-11.7036,18.7961-18.6695.1646-4.2044-3.5808-8.0622-2.0773-12.0538,2.2343-5.933,12.7128-4.8986,14.5344-10.0952,1.4175-4.0443-3.16-9.7143-10.0647-15.6265a11.1555,11.1555,0,0,1-2.1695-5.8271,11.4732,11.4732,0,0,1,3.1657-8.6076,8.9079,8.9079,0,0,0,9.2321-12.9883,15.7709,15.7709,0,0,0,6.7141-26.8971c5.2415-5.0178,10.6975-10.1988,15.69-14.8747,5.8684-5.4964,13.331-12.6566,11.4273-17.7856-1.6836-4.5371-9.914-4.5035-14.5318-10.5144a17.7555,17.7555,0,0,1-2.7819-5.3336c27.0322-2.4494,48.71-.5435,60.0435,8.7058,1.9906,1.6245,6.2673,5.5995,11.75,5.181,7.3117-.5582,12.2452-10.167,16.0642-8.8962,3.4253,1.14,1.8609,9.5,5.4448,11.7726,5.8578,3.7144,20.8489-9.0091,34.9365-23.0829q50.3865,4.8687,100.773,9.7377a447.9452,447.9452,0,0,0-54.4778,370.5475,477.2791,477.2791,0,0,0-67.0681,316.0563,138.4052,138.4052,0,0,0,92.7554,73.7541c12.8245-30.9878,24.1644-62.2124,34.2078-94.1939,7.5385-24.005,14.149-47.7691,20.0914-72.2241a225.0971,225.0971,0,0,0,86.13-181.4462,153.39,153.39,0,0,0,33.1446-99.9017A248.74,248.74,0,0,0,867.749,262.2321a58.795,58.795,0,0,0,47.69-11.6416q-.4342,17.0121-.8691,34.0241c12.9979-.6754,28.6676-.8827,29.3709,2.6051.545,2.7025-9.8012,6.6627-15.5541,14.5444a30.19,30.19,0,0,0-5.9734,13.8986c-1.2477,9.2589,2.5132,18.393,8.9923,25.5784-6.1557,2.9236-10.6845,6.3664-10.5933,10.045.1655,6.6751,15.4137,11.9369,20.3771,13.3815,4.9521,1.4409,11.7028,2.7491,15.8762,8.422C959.1791,375.9626,959.5561,378.222,959.1754,379.4692Z"
                        /></svg
                    >
                {/if}
            {/each}
        </div>
    {/each}
</div>

{#if minutes.at(-1) === "1"}
    <p class="footer" transition:fade>
        Made with <svg
            viewBox="0 0 1792 1792"
            preserveAspectRatio="xMidYMid meet"
            xmlns="http://www.w3.org/2000/svg"
            ><path
                class="heart"
                class:odd_heart
                d="M896 1664q-26 0-44-18l-624-602q-10-8-27.5-26T145 952.5 77 855 23.5 734 0 596q0-220 127-344t351-124q62 0 126.5 21.5t120 58T820 276t76 68q36-36 76-68t95.5-68.5 120-58T1314 128q224 0 351 124t127 344q0 221-229 450l-623 600q-18 18-44 18z"
            ></path></svg
        > by RootM
    </p>
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
        font-family: "Roboto Mono";
        src: url("/RobotoMono-VariableFont_wght.ttf");
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

    .circle {
        width: min($diameter, $diameter-height-breakpoint);
        height: $diameter;
        max-height: $diameter-height-breakpoint;
        margin: 5px;
        //background-color: $inactive;
        fill: $inactive;
        transition: 0.1s;

        display: flex;
        justify-content: center;
        vertical-align: middle;
        align-items: center;
        color: $inactive;
    }

    .circle.secondary.show_inactive_numbers {
        color: $secondary;
    }

    .Badger_Color {
        fill: $inactive;
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
            height: min(
                $diameter-vertical,
                $diameter-height-breakpoint-vertical
            );
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
                transform: translate($random-x, -60px)
                    scale($random-scale)
                    rotate($rotate);
                animation: fall-#{$i}
                    $fall-duration
                    $fall-delay
                    linear
                    infinite;
            }

            @keyframes fall-#{$i} {
                #{percentage($random-yoyo-time)} {
                    transform: translate($random-x-end, $random-yoyo-y)
                        scale($random-scale);
                }

                to {
                    transform: translate($random-x-end-yoyo, 100vh)
                        scale($random-scale) rotate($rotate);
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
