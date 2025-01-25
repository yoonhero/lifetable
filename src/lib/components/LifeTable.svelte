<!-- yellow/green/red -->
<script lang="ts">
    import { error } from "@sveltejs/kit";
    import { onMount } from "svelte";
    import { parse } from "svelte/compiler";

    // graph style
    interface focusData {
        id: string;
        color: "empty" | "green" | "yellow" | "red";
        parentId: string | undefined;
        childrenIds: string[];
    }

    type focusDatas = {
        [key: string]: focusData;
    };

    const ROOT = "root";
    const YEAR = "year";
    const WEEK = "week";
    const DAY = "day";
    const keys = [ROOT, YEAR, WEEK, DAY];

    function getTypeOfId(id: string) {
        const splitted = id.split("-");
        return keys[splitted.length - 1];
    }

    function draw(parentId: string) {
        const type = getTypeOfId(parentId);
        let currentParentItem: focusData = items[parentId];
        let childrens;
        let splitted = parentId.split("-");
        splitted.shift();
        const parentIdWihoutRoot = splitted.join("-");
        switch (type) {
            case ROOT:
                // const now = new Date(Date.now());
                // const thisYear = now.getFullYear();
                childrens = createChildrens(84, parentId, 2025);
                break;
            case YEAR:
                const totalWeeksOfYear = getTotalWeeksOfYear(parentIdWihoutRoot);
                childrens = createChildrens(totalWeeksOfYear, parentId, 1);
                break;
            case WEEK:
                const totalDaysOfWeek = getTotalDaysOfWeek(parentIdWihoutRoot);
                childrens = createChildrens(totalDaysOfWeek, parentId, 1);
                break;
            default:
                return;
        }
        currentParentItem.childrenIds = childrens.map((v) => v.id);
        items[parentId] = currentParentItem;
    }

    function leapYear(year: number) {
        return new Date(year, 1, 29).getDate() === 29;
    }

    function getTotalWeeksOfYear(parentId: string) {
        const year = parseInt(parentId);
        const firstDay = new Date(year, 0, 1);
        const lastDay = new Date(year, 11, 31);
        const firstDayNumber = firstDay.getDay();
        const lastDayNumber = lastDay.getDay(); // 0-6 sun-sat
        let totalDay = 365;
        if (leapYear(year)) {
            totalDay = 366;
        }
        const totalWeeksOfYear = (totalDay - (7 - firstDayNumber) - (lastDayNumber + 1)) / 7 + 2;
        return totalWeeksOfYear;
    }
    function getTotalDaysOfWeek(parentId: string) {
        const year = parseInt(parentId.split("-")[0]);
        const week = parseInt(parentId.split("-")[1]);
        if (week == 1) {
            const firstDay = new Date(year, 0, 1);
            const firstDayNumber = firstDay.getDay();
            return 7 - firstDayNumber;
        } else if (week == getTotalWeeksOfYear(`${year}`)) {
            const lastDay = new Date(year, 11, 31);
            const lastDayNumber = lastDay.getDay();
            return lastDayNumber + 1;
        }
        return 7;
    }

    function createChildrens(n: number, parentId: string, start: number = 0): focusData[] {
        return Array.from({ length: n }, (_, k) => {
            const currentId = `${parentId}-${start + k}`;
            const data: focusData = {
                id: currentId,
                color: "empty",
                parentId: parentId,
                childrenIds: [],
            };
            items[currentId] = data;
            return data;
        });
    }

    let items: focusDatas = {};
    let currentFocusDatas: focusData[] = [];
    let deepest: boolean = false;
    let currentParentType: string;
    let currentParentId: string;
    const colorMap = { empty: "transparent", green: "bg-green-400", yellow: "bg-yellow-400", red: "bg-red-400" };
    let isLastWeek = false;
    let isFirstWeek = false;
    const weekTable = ["SUN", "MON", "TUE", "WED", "TUR", "FRI", "SAT"];
    let errorMsg: string;

    $: if (deepest) {
        isFirstWeek = currentParentId.split("-")[2] == "1";
        isLastWeek = parseInt(currentParentId.split("-")[2]) == getTotalWeeksOfYear(currentParentId.split("-")[1]);
    }

    onMount(() => {
        const root: focusData = { id: "root", color: "empty", childrenIds: [], parentId: undefined };
        items["root"] = root;
        draw("root");
        currentParentId = "root";
    });

    $: if (currentParentId) {
        currentParentType = getTypeOfId(currentParentId);
    }

    $: if (currentParentId && items[currentParentId].childrenIds.length > 0) {
        const parent = items[currentParentId];
        currentFocusDatas = parent.childrenIds.map((item) => {
            return items[item];
        });
    }

    function getToday() {
        let today = new Date(Date.now());
        today.setMilliseconds(0);
        today.setSeconds(0);
        today.setMinutes(0);
        today.setHours(0);
        return today;
    }

    function isUpdatePossible(targetId: string): boolean {
        const parsed = parseCallender(targetId);
        const today = getToday();
        return today.getTime() <= parsed.getTime() && today.setDate(today.getDate() + 1) >= parsed.getTime();
    }

    function cycleColor(targetId: string) {
        if (!isUpdatePossible(targetId)) {
            errorMsg = "오늘만 기록하실 수 있습니다.";
            return;
        }
        errorMsg = "";

        let data = items[targetId];
        const cycleOrder: focusData["color"][] = ["empty", "green", "red"];
        const currentIndex = cycleOrder.indexOf(data.color);
        if (currentIndex === -1 || currentIndex === cycleOrder.length - 1) {
            data.color = cycleOrder[0];
        } else {
            data.color = cycleOrder[currentIndex + 1];
        }
        items[targetId] = data;
        currentFocusDatas = [...currentFocusDatas];
        updateColor(data.parentId);
    }

    function updateColor(id: string) {
        const data = items[id];
        const childrenIds = data.childrenIds;

        const totalGreen = childrenIds.filter((v) => items[v].color == "green").length;
        const totalRed = childrenIds.filter((v) => items[v].color == "red").length;

        if (Math.max(totalGreen, totalRed) > 0) {
            if (totalGreen > totalRed) {
                data.color = "green";
            } else if (totalGreen < totalRed) {
                data.color = "red";
            } else {
                data.color = "yellow";
            }
        } else {
            data.color = "empty";
        }

        if (data.parentId) {
            return updateColor(data.parentId);
        }
    }

    function focus(targetId: string) {
        const data = items[targetId];
        // if (on.childrenIds.length == 0) {
        const type = getTypeOfId(data.id);

        switch (type) {
            case DAY:
                return;
            case WEEK:
                deepest = true;
                break;
            default:
                deepest = false;
        }
        currentParentId = data.id;
        if (data.childrenIds.length == 0) {
            draw(currentParentId);
        }
    }

    function outFocus(id: string) {
        const type = getTypeOfId(id);

        switch (type) {
            case ROOT:
                return;
            case WEEK:
                deepest = false;
            default:
                currentParentId = items[id].parentId;
        }
    }

    function format(label: string, i: number) {
        // if (!id) return [];
        // let label = "";
        switch (keys[i]) {
            case ROOT:
                return "Life";
            case DAY:
                return `${label}번째 날`;
            case WEEK:
                return `${label}번째 주`;
            case YEAR:
                return `${label}년`;
        }
    }

    function parseCallender(id: string) {
        const type = getTypeOfId(id);
        const splitted = id.split("-");

        const curYear = parseInt(splitted[1]);
        let result = new Date(curYear, 0, 1);
        switch (type) {
            case DAY:
                const curDay = parseInt(splitted[3]);
                result.setDate(result.getDate() + curDay - 1);
            case WEEK:
                const curWeek = parseInt(splitted[2]);
                if (curWeek < 2) break;
                result.setDate(result.getDate() + (curWeek - 2) * 7 + getTotalDaysOfWeek(`${curYear}-1`));
        }
        return result;
    }

    function isDisabled(id: string): boolean {
        const today = Date.now();
        // const today = new Date(2026, 0, 3);
        const parsed = parseCallender(id);
        return today < parsed.getTime();
    }

    function handleWheel(event: WheelEvent) {
        const today = getToday();
        let focusKey;
        for (const focusData of currentFocusDatas) {
            if (parseCallender(focusData.id).getTime() <= today.getTime()) {
                focusKey = focusData.id;
            }
        }
        // If user scrolled up => zoom in
        if (event.deltaY < 0) {
            if (currentParentType == WEEK) {
                return;
            }
            focus(focusKey);
        }
        // If user scrolled down => zoom out
        else {
            outFocus(currentParentId);
        }
    }
</script>

<div class="relative w-full max-w-[756px] min-h-[300px] flex flex-col items-center" on:wheel|passive={handleWheel}>
    {#if currentParentId}
        <div class="absolute top-[-50px] font-md text-2xl divide-x-0">
            {#each currentParentId.split("-") as label, i}
                <a
                    class="px-2 cursor-pointer select-none text-gray-800 font-bold"
                    on:click={() =>
                        focus(
                            currentParentId
                                .split("-")
                                .slice(0, i + 1)
                                .join("-")
                        )}>{format(label, i)}</a
                >
            {/each}
        </div>
    {/if}
    <div class="relative {deepest ? 'flex flex-row' : 'block'}">
        {#if isFirstWeek}
            {#each Array.from({ length: 7 - currentFocusDatas.length }) as _, i}
                <div class="flex flex-col text-center text-sm text-gray-500">
                    <span>{weekTable[i]}</span>
                    <button disabled={true} class="dayBox bg-gray-600 opacity-10"></button>
                </div>
            {/each}
        {/if}
        {#each currentFocusDatas as focusData, i}
            {#if !deepest}
                <div class="inline-block">
                    <button
                        class="box {colorMap[focusData.color]} {focusData.color == 'empty' &&
                            'border-2'} text-sm border-gray-300 disabled:bg-gray-900 disabled:opacity-20 font-light"
                        on:click={() => focus(focusData.id)}
                        disabled={isDisabled(focusData.id)}
                    ></button>
                </div>
            {:else}
                <div class="flex flex-col text-center text-sm text-gray-500">
                    <span>{!isFirstWeek ? weekTable[i] : weekTable[7 - currentFocusDatas.length + i]}</span>
                    <button
                        class="dayBox {colorMap[focusData.color]} {focusData.color == 'empty' &&
                            'border-2'} border-gray-300 disabled:bg-gray-900 disabled:opacity-20"
                        on:click={() => cycleColor(focusData.id)}
                        disabled={isDisabled(focusData.id)}
                    ></button>
                </div>
            {/if}
        {/each}
        {#if isLastWeek}
            {#each Array.from({ length: 7 - currentFocusDatas.length }) as _, i}
                <div class="flex flex-col text-center text-sm text-gray-500">
                    <span>{weekTable[currentFocusDatas.length + i]}</span>
                    <button disabled={true} class="dayBox bg-gray-600 opacity-10"></button>
                </div>
            {/each}
        {/if}
    </div>
    {#if errorMsg}
        <div class="mt-4">
            <span class="text-sm text-red-400">{errorMsg}</span>
        </div>
    {/if}
</div>

<style>
    .box {
        margin: 2px;
        width: 50px;
        height: 50px;

        border-radius: 10px;
    }

    .dayBox {
        margin: 2px;
        width: 80px;
        height: 80px;
        border-radius: 15px;
    }
</style>
