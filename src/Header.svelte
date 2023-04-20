<script>
    import { onMount } from 'svelte';
    import { createEventDispatcher } from 'svelte';
    import { branchList } from './branchList';

    // コンポーネント共通変数
    const dispatch = createEventDispatcher();
    let filteredOffices        = [];
    let filteredBranches       = [];
    let show                   = '';

    onMount(() => {
        //headerの高さ分、mainのコンテンツを下げる（clear-fixの高さを指定する）
        let headerHeight      = document.getElementById('header').clientHeight;
        const clearFix        = document.getElementById('clear-fix');
        clearFix.style.height = headerHeight + 'px';

        // leftNavのwidthを取得し、支店候補のドロワーのleftを設定、取得済みのheaderの高さを使用し、支店候補ドロワーのtopを設定
        const leftMenu              = document.querySelector('#left-menu');
        const navWidth              = leftMenu.clientWidth;
        const drawerBranchList      = document.getElementById('js-branch-list');
        drawerBranchList.style.left = navWidth + 'px';
        drawerBranchList.style.top  = headerHeight + 'px';
    });

    // マウスオーバーで表示されるドロワリスト　(branchと言うワードに本部が含まれる場合と含まれない場合があってわかりにくい)
    // 例えば全体をsection(部門), 区別するときはoffice, branch(本部、支店)などと分けたら良いかもだけど、大がかりだな。。
    const handleKanaHover = (event) => {
        showBranchList(event.target.textContent);
    }
    function showBranchList(kanaIndex) {
        // 頭文字からリストに表示される本部と支店を抽出
        const orderedOfficesAndBranchesObject = generateOrderedBranchLists();
        filteredOffices  = orderedOfficesAndBranchesObject.offices.filter(office => office.representativeVal === kanaIndex);
        filteredBranches = orderedOfficesAndBranchesObject.branches.filter(branch => branch.representativeVal === kanaIndex);
        
        // マウスオーバーで表示されるドロワリスト全体の高さを制御（ハードコーディング気味）
        // const listHeadHeight  = document.querySelector('.branch-list-head').offsetHeight;
        // const eachLiTagHeight = document.querySelector('.branch-list-li').offsetHeight;
        // console.log(listHeadHeight, eachLiTagHeight)
        const eachLiTagHeight = 45;
        const listHeadHeight  = 59.5 * 2 //'本部'、'支店'の高さ
        let listHeight = eachLiTagHeight * (filteredOffices.length + filteredBranches.length) + listHeadHeight;
        document.querySelector('.branch-list').style.height = listHeight + 'px';
        show = 'show';
    }

    // branchList.jsから、あいうえお順に本部arrayと営業店arrayを作って、オブジェクトに格納して返す。
    function generateOrderedBranchLists() {
        const orderedHeadOffices = branchList[0].branches.sort((a, b) => a.firstLetter.localeCompare(b.firstLetter));
        let onlyBranches         = [];
        branchList.slice(1).map(area => area.branches.map(branch => onlyBranches.push(branch)));
        const orderedBranches    = onlyBranches.sort((a, b) => a.firstLetter.localeCompare(b.firstLetter));
        
        const orderedOfficesAndBranchesObject = {
            offices: orderedHeadOffices,
            branches: orderedBranches
        }
        return orderedOfficesAndBranchesObject;
    }

    const closeBranchList = () => {
        show = '';
        document.querySelector('.branch-list').style.height = 0;
    }

    // 部門がクリックされたら、
    const handleClickBranch = (event) => {
        dispatch('listClick', {targetBranch: event.target.innerText});
        closeBranchList();
    }
    const handleKeydownBranch = (event) => {
        console.log(event.target.innerText);
    }

    // logoクリックで初期表示に戻る
    const handleLogoClick = () => {
        dispatch('logoClick');
    }
    
    // kanaIndexをeachで作成
    const kanaOrigin   = "あかさたなはまやらわ";
    const arrKanaIndex = kanaOrigin.split('');

    const void0 = () => {
        return;
    }
</script>

<header id="header">
    <div class="header-log" on:click={handleLogoClick} on:keydown={void0} on:mouseover={closeBranchList} on:focus={void0}>Extinction TEL</div>
    <ul class="kana-index">
        {#each arrKanaIndex as kana}
            <li class="kana" on:mouseover={handleKanaHover} on:focus={handleKanaHover}>{kana}</li>
        {/each}
        <div class="list-closer" on:mouseover={closeBranchList} on:focus={void0}></div>
    </ul>
</header>
<div class="clear-fix" id="clear-fix"></div>
<section class="branch-list {show}" id='js-branch-list' on:mouseleave={closeBranchList}>
    <div>
        <p class="branch-list-head">本部</p>
        <ul class="branch-list-ul">
            {#each filteredOffices as office}
                <li class="branch-list-li" on:click={handleClickBranch} on:keydown={handleKeydownBranch}>{office.branchName}</li>
            {/each}
        </ul>
    </div>
    <div>
        <p class="branch-list-head">支店</p>
        <ul class="branch-list-ul">
            {#each filteredBranches as branch}
                <li class="branch-list-li" on:click={handleClickBranch} on:keydown={handleKeydownBranch}>{branch.branchName}</li>
            {/each}
        </ul>
    </div>
</section>

<style>
    header {
        display: flex;
        justify-content: start;
        align-items: center;
        position: fixed;
        top: 0;
        left: 0;
        width: 100vw;
        height: 60px;
        background-color: #fff;
        z-index: 990;
        box-shadow: 0 3px 20px 3px rgba(0, 0, 0, .2);
    }
    .header-log {
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100%;
        width: clamp(310px, 1rem + 15%, 500px);
        flex-shrink: 0;
        font-size: 2.5rem;
        font-weight: bold;
        cursor: pointer;
        color: #4b8fb8;
    }
    .header-log:hover {
        opacity: 0.7;
    }
    .kana-index {
        display: flex;
        justify-content: start;
        list-style: none;
        width: 100%;
        height: 100%;
    }
    .kana {
        display: flex;
        justify-content: center;
        align-items: center;
        width: 10%;
        max-width: 80px;
        height: 100%;
        text-align: center;
        cursor: pointer;
        transition: all .3s cubic-bezier(.16,.41,.68,.91);
    }
    .kana:hover {
        background-color: #eee;
    }
    .list-closer {
        width: 80px;
        height: 60px;
    }

    /* ドロワーリスト */
    .branch-list {
        background-color: #eee;
        position: absolute;
        top: 60px;
        left: 250px;
        z-index: 990;
        width: 80%;
        max-width: 800px;
        height: 0;
        overflow-y: scroll;
        transition: all .3s cubic-bezier(.16,.41,.68,.91);
        box-shadow: 0 3px 20px 3px rgba(0, 0, 0, .2);
    }
    .branch-list.show {
        height: 100%;
        max-height: calc(100vh - 60px);
    }
    .branch-list-head {
        margin: 0;
        padding: 17px 1rem;
        font-weight: bold;
        font-size: 1.7rem;

    }
    .branch-list-ul {
        list-style: none;
    }
    .branch-list-li {
        box-sizing: border-box;
        cursor: pointer;
        padding: 10px 10px 10px 30px;
        border-bottom: 1px solid #ccc;
    }
    .branch-list-li:hover {
        background-color: #ddd;
    }
</style>