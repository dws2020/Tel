<script>
    // import { jsEmpList } from '../public/empList.js';
    import { createEventDispatcher } from 'svelte';
    export let belongs;

    // コンポーネント内共通変数
    const dispatch       = createEventDispatcher();
    const orderedEmpList = jsEmpList.sort((a, b) => a.kana.localeCompare(b.kana));
    let empOfBelongs     = [];
    $: empOfBelongs      = orderedEmpList.filter(emp => emp.belongs === belongs);

    // イベントハンドラー
    const handleEmpClick = (event) => {
        // メンバーリストクリックで、選択したメンバー情報を渡して右メニュー表示。
        dispatch('empListClick', {targetEmp: event.currentTarget});
    }
    const keydownCancel = () => {
        return;
    }
    
    // 関数 
    /**
     * フリガナ一文字目（半角カナ）の子音a列が同じか比較する
     * @param {string} kana1
     * @param {string} kana2
     * @returns {bool} 
     */
    function isSameInitialKana(kana1, kana2) {
        return toHeadKana(getInitialKana(kana1)) === toHeadKana(getInitialKana(kana2)); 
    }
    // フリガナ一文字目（半角カナ）を取得
    function getInitialKana(kana) {
        let arrKana = kana.split('');
        return arrKana[0];
    }
    // フリガナ一文字目（半角カナ）を子音a列（全角カナ）に変換（例：'ｴ' -> 'ア'）
    function toHeadKana(targetLetter) {
        const AllKana = [
            {head: "ア", body: ["ｱ", "ｲ", "ｳ", "ｴ", "ｵ"]},
            {head: "カ", body: ["ｶ", "ｷ", "ｸ", "ｹ", "ｺ"]},
            {head: "サ", body: ["ｻ", "ｼ", "ｽ", "ｾ", "ｿ"]},
            {head: "タ", body: ["ﾀ", "ﾁ", "ﾂ", "ﾃ", "ﾄ"]},
            {head: "ナ", body: ["ﾅ", "ﾆ", "ﾇ", "ﾈ", "ﾉ"]},
            {head: "ハ", body: ["ﾊ", "ﾋ", "ﾌ", "ﾍ", "ﾎ"]},
            {head: "マ", body: ["ﾏ", "ﾐ", "ﾑ", "ﾒ", "ﾓ"]},
            {head: "ヤ", body: ["ﾔ", "ﾕ", "ﾖ"]},
            {head: "ラ", body: ["ﾗ", "ﾘ", "ﾙ", "ﾚ", "ﾛ"]},
            {head: "ワ", body: ["ﾜ", "ｦ", "ﾝ"]}
        ]
        let resultArray = AllKana.filter(objKana => objKana.body.includes(targetLetter));
        return resultArray[0].head;
    }

</script>

<div class="emp-cards-area">
    {#if belongs}
        <ul class="emp-cards">
        {#each empOfBelongs as emp, i}
            <li class="emp-card" on:click|stopPropagation={handleEmpClick} on:keydown={keydownCancel}>
                {#if i !== 0}
                    {#if isSameInitialKana(empOfBelongs[i-1].kana, empOfBelongs[i].kana)}
                        <div class="emp-initial"></div>
                    {:else}
                        <div class="emp-initial">{toHeadKana(getInitialKana(empOfBelongs[i].kana))}</div>
                    {/if}
                {:else if i === 0}
                    <div class="emp-initial">{toHeadKana(getInitialKana(empOfBelongs[i].kana))}</div>
                {/if}
                <div class="photo card-content">NO PHOTO</div>
                <div class="emp-name card-content">{emp.name}</div>
                <div class="emp-no card-content">{emp.empNo}</div>
                <div class="kana card-content">{emp.kana}</div>
                <div class="belongs card-content">{emp.belongs}</div>
                <div class="position card-content">{emp.position}</div>
            </li>
        {/each}
        </ul>
    {/if}
</div>

<style>
    .emp-cards-area {
        display: flex;
        width: 100%;
        height: calc(100vh - 60px);
        overflow-y: scroll;
        /*スクロールバー非表示（IE・Edge）*/
        -ms-overflow-style: none;
    }
    .emp-cards-area::-webkit-scrollbar {
        display: none;
    }
    .emp-cards {
        display: flex;
        flex-flow: column nowrap;
        align-items: center;
        list-style: none;
        margin: 30px 0 0 0;
        width: 100%;
    }
    .emp-cards::after {
        content: '';
        padding: 20px;
    }
    .emp-card {
        position: relative;
        display: grid;
        grid-template-rows: 4.5rem 1.6rem 1.6rem 4.5rem; 
        grid-template-columns: 150px 10rem 1fr;
        background-color: #fff;
        width: 601px;
        border-radius: 5px;
        box-shadow: 3px 3px 10px -1px rgb(0 0 0 / 20%);
        margin-bottom: 20px;
        padding: 10px;
        cursor: pointer;
        color: #444;
        transition: all .1s cubic-bezier(.16,.41,.68,.91);
    }
    .emp-card:hover {
        opacity: 0.7;
        transform: scale(1.02);
        box-shadow: 3px 3px 15px 1px rgb(0 0 0 / 15%);
    }
    .card-content {
        display: flex;
        justify-content: flex-start;
        align-items: center;
    }
    .emp-initial {
        position: absolute;
        font-size: 2.0rem;
        top: 5px;
        left: -3rem;
    }
    .photo {
        display: flex;
        justify-content: center;
        color: #aaa;
        grid-row: 1/5;
        grid-column: 1;
    }
    .emp-name {
        grid-row: 1;
        grid-column: 2/4;
        font-size: 2rem;
        font-weight: bold;
    }
    .emp-no {
        grid-row: 2;
        grid-column: 2/4;
    }
    .kana {
        grid-row: 3;
        grid-column: 2/4;
    }
    .belongs {
        font-size: 1.6rem;
        grid-row: 4;
        grid-column: 2;
    }
    .position {
        font-size: 1.6rem;
        grid-row: 4;
        grid-column: 3;
    }
</style>