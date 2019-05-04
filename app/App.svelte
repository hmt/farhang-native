<page actionBarHidden="true" class="page">
    <stackLayout padding="5">
        <label text="Farhang.im durchsuchen"></label>
        <textField bind:text="{term}" hint="Hier deutsch oder persisch eingeben" />
        <listView items="{lemmas}" style="height:100%" on:itemTap="{showLemma}">
            <Template let:item>
                {#if item.id === selectedItem.id}
                    {#each translations as t}
                        <flexboxLayout>
                            <label alignSelf="flex-start" flexGrow="1" class="list-item active" text="{`${persian ? t.target : t.source}`}" textWrap="true"/>
                            <label alignSelf="flex-end" flexGrow="1" class="list-item active" text="{`${persian ? t.source : t.target}`}" textWrap="true"/>
                        </flexboxLayout>
                    {/each}
                {:else}
                    <label class="list-item" text="{item.lemma}" textWrap="true" />
                {/if}
            </Template>
        </listView>
    </stackLayout>
</page>

<script>
    import { Template } from 'svelte-native/components';
    import * as http from 'tns-core-modules/http'

    let term = ''
    let lemmas = []
    let translations
    let selectedItem = {}
    let persian
    $:  if (term.length > 1) {
        persian = checkPersian(term)
        const normalized = persian ? term.replace(/([\u064B-\u0655])/g, '') : term
        const uri = encodeURI(`https://api.farhang.im/a/${normalized}`)
        http.getJSON(uri)
            .then(res => setLemmas(res))
    }
    $: if (term.length === 0) lemmas = []
    function setLemmas(res) {
        lemmas = res
    }
    function showLemma (lemma) {
        selectedItem = lemma.view.bindingContext;
        const uri = encodeURI(`https://api.farhang.im/g/${selectedItem.id}`)
        http.getJSON(uri)
            .then(res => {
                translations = res
                lemmas = Array.from(lemmas)
            })
    }
    function checkPersian (string) {
        const regex = /[\u0622\u0627\u0628\u067E\u062A-\u0632\u0686\u0698\u0633-\u063A\u0641\u0642\u06A9\u06AF\u0644-\u0648\u06CC]/
        return regex.test(string)
    }
</script>

<style>
    page {
        background-color: black;
    }
    textField {
        font-size: 20;
        color:white;
        placeholder-color: gray;
        margin: 5;
    }
    label {
        color: white;
    }
    .list-item {
        font-size: 20;
        color: white;
        /* margin-left: 20; */
        padding-top: 5;
        padding-bottom: 10;
    }
    .list-item.active {
        font-weight: bold;
        color: white;
    }
</style>