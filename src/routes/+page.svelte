<script lang="ts">
    import { onMount } from "svelte";

    let pi: string;
    let index: number = 0;
    const spacing = 100;
    let lossCharacter: string = '';
    let lost: boolean = false;

    let digitInput: HTMLInputElement | undefined;

    function reset() {
        index = 0;
        lossCharacter = ' ';
        lost = false;
    }

    $: if(digitInput) {
        digitInput?.select();
    }

    onMount(async () => {
        pi = await (await fetch('/pi/pi_1_million.txt')).text();
        reset();
    });

</script>

<style lang="scss">

    * {
        @apply font-segoe;
    }

    .number-input::-webkit-outer-spin-button,
    .number-input::-webkit-inner-spin-button {
        -webkit-appearance: none;
        appearance: none;
        margin: 0;
    }

</style>

<svelte:body
    on:keypress={ev => {
        if(ev.code == 'KeyR') {
            reset();
        }
    }}
/>

{#if pi}
    <div class="w-full h-screen flex gap-16 flex-col justify-center items-center overflow-clip">
        <div class="h-18 text-zinc-500 text-6xl font-bold relative flex justify-center items-center">
            <!-- The text-xl is to get around a space character being added between text. -->
            <div class="absolute right-full text-nowrap gap-0 text-xs">
                {#if index < spacing}
                    <span class="text-6xl">3.</span>
                {/if}
                <span class="text-6xl">
                    {pi.slice(Math.max(index - spacing, 0), index)}
                </span>
            </div>
            {#if !lost}
                <input
                    bind:this={digitInput}
                    type="number" step="1" min="0" max="9"
                    class="w-12 h-16 number-input bg-transparent text-white border-b-4 border-b-white"
                    on:input={ev => {

                        const input = ev.currentTarget.value;
                        ev.currentTarget.value = '';
                        ev.preventDefault();

                        if(lost) return;

                        if((/^\d$/.test(input))) {
                            if(pi[index] == input) {
                                index++;
                            } else {
                                lossCharacter = input;
                                lost = true;
                            }
                        }
                    }}
                >
            {:else}
                <span class="border-b-4 border-b-red-500 text-red-500">
                    {lossCharacter}
                </span>
                <span class="fixed -translate-y-full text-emerald-600">
                    {pi[index]}
                </span>
                <span class="absolute left-full">
                    {pi.slice(index + 1, index + spacing)}
                </span>
            {/if}
        </div>
        <div class="flex justify-center items-center gap-8">
            <span class="text-white text-4xl font-bold">
                {index} digits 
            </span>
            <button
                class="px-8 py-4 bg-zinc-300 hover:bg-zinc-400 rounded-2xl text-2xl text-black font-bold"
                on:click={() => reset()}
            >Retry</button>
        </div>
    </div>
{/if}
