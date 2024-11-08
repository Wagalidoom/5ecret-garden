<script lang="ts">
    import {onMount} from 'svelte';
    import IMask from 'imask';
    import type {TokenBalanceRow} from "@circles-sdk/data";
    import Avatar from "$lib/components/Avatar.svelte";
    import {tokenTypeToString} from "$lib/pages/SelectAsset.svelte";
    import {roundToDecimals} from "$lib/utils/shared";

    export let balanceRow: TokenBalanceRow;
    export let amount: number = 0;
    export let maxAmountCircles: number = -1;
    export let staticCircles: number = 0;

    let inputElement: HTMLInputElement;
    let avatarWidth: string = "12rem";

    const maskOptions = {
        mask: Number,
        scale: 2,
        signed: false,
        thousandsSeparator: ',',
        padFractionalZeros: true,
        normalizeZeros: true,
        radix: '.',
        mapToRadix: ['.'],
        placeholderChar: '_',  // Placeholder character
    };

    function set() {
        avatarWidth = (12.5 - (inputElement.value.length * 0.6)) + "rem";
        amount = parseFloat(inputElement.value.replace(/,/g, '')) || 0;
        console.log("Amount set to:", amount);
    }

    function setMaxAmount() {
        amount = maxAmountCircles >= 0 ? roundToDecimals(maxAmountCircles) : roundToDecimals(balanceRow.circles);
        inputElement.value = amount.toString();
        set();
    }

    onMount(() => {
        if (amount > 0) {
            setTimeout(() => {
                if (amount > balanceRow.circles) {
                    amount = roundToDecimals(balanceRow.circles);
                }
                inputElement.value = amount.toString();
                set();
                IMask(inputElement, maskOptions);
            }, 0);
        }

        // Subscribe to all possible value changes of the text input. Should be called whenever the input changes.
        inputElement.oninput = () => {
            set();
        };
        inputElement.onblur = () => {
            setTimeout(set, 0); // Delay to allow for any potential input changes
        };

        IMask(inputElement, maskOptions);
    });
</script>
<style>
    .fade-out-gradient {
        background: linear-gradient(to right, rgba(255, 255, 255, 0) 0%, rgba(255, 255, 255, 1) 100%);
    }
</style>

<div>
    <div class="relative">
        <input type="text"
               placeholder="0.00"
               class="input input-lg w-full text-2xl font-light placeholder-gray-400 focus:outline-none focus-visible:outline-none focus:ring-0"
               bind:this={inputElement}
               style="letter-spacing: 0.1em;"
        />

        <div class="absolute inset-y-0 right-0 flex items-center overflow-hidden" style="width:{avatarWidth}">
            <div class="min-w-96 relative flex items-center">
                <Avatar address={balanceRow?.tokenOwner} clickable={false}>
                    {tokenTypeToString(balanceRow?.tokenType)}
                </Avatar>
            </div>
        </div>
        <div class="absolute top-0 right-0 bottom-0 w-12 m-1 pointer-events-none fade-out-gradient rounded"
             style="margin-right: 0.1rem;"></div>
    </div>
    {#if balanceRow.isInflationary}
<!--        <div class="ml-6 text-gray-400">10.00 static Circles</div>-->
    {/if}
    <p class="ml-6">
        <button class="btn btn-sm mt-4 font-normal" on:click={setMaxAmount}>
            {#if maxAmountCircles == -1}
                <span class="loading loading-spinner text-primary"></span>
            {:else if maxAmountCircles == -2}
                ⚠️
            {/if}
            Max <span
                class="font-medium">{maxAmountCircles >= 0 ? roundToDecimals(maxAmountCircles) : roundToDecimals(balanceRow?.circles)}</span>
        </button>
    </p>
</div>