<script>
    'use strict';
    import { createEventDispatcher, onMount } from "svelte";

    // コンポーネント共通変数
    const dispatch = createEventDispatcher();
    let callerBelongs;
    let callerName;
    let arrHistoryBelongs = [];
    let arrHistoryName    = [];

    onMount( ()=> {
        renderHistory();
    });

    const handleMemoBtnClick = event => {
        if ((callerBelongs === '' && callerName === '')||(!callerBelongs && !callerName)) return;
        setHistory(callerBelongs, callerName);

        // 社内リストを選択した時の処理に合わせるために、親子構造のエレメント作成
        const parent = document.createElement('div');
        parent.classList.add('parent');
        const numOfChildElements = 7;
        for (let i = 0; i < numOfChildElements; i++) {
            const child = document.createElement('div');
            child.classList.add(`child${i}`);
            switch (i) {
                case 2:
                    child.innerText = callerName;
                    break;
                case 5:
                    child.innerText = callerBelongs;
                    break;
                default:
            }
            parent.append(child);
        }
        dispatch('createMemo', {targetEmp: parent});
        clearInput();
        callerBelongs = '';
        callerName    = '';
    }

    // 左メニューの外部電話のインプット欄をクリア
    function clearInput() {
        document.getElementById('caller-belongs').value = '';
        document.getElementById('caller-name').value    = '';
    }

    // 履歴管理
    function renderHistory() {
        let arrHistory    = getHistoriesFromLocalStorage();
        arrHistory.reverse();
        if (!arrHistory) return;
        arrHistoryBelongs = arrHistory.map(arr => arr.belongs); 
        arrHistoryName    = arrHistory.map(arr => arr.name);
    }

    function setHistory(callerBelongs, callerName) {
        let histories = getHistoriesFromLocalStorage() || [];
        console.log(histories);
        const maxHistory = 10;
        const newHistory = {belongs: callerBelongs, name: callerName}
        histories = deleteDuplication(histories, newHistory);
        histories.push(newHistory);
        // localStorageが１０個以上あったら、一番古いものを削除。
        if (histories.length >= maxHistory) {
            histories.shift();
        }
        setHistoriesToLocalStorage(histories);
        renderHistory();
    }

    function deleteDuplication(histories, newHistory) {
        for (let i = 0; i < histories.length; i++) {
            if (histories[i].belongs == newHistory.belongs && histories[i].name == newHistory.name) {
                histories.splice(i, 1);
                break;
            }
        }
        return histories;
    }

    /**
     * 履歴配列を受け取りJSONに変換て、ローカルストレージに保存する
     * @param {array} arrHistory
     */
    function setHistoriesToLocalStorage(arrHistory) {
        const json = JSON.stringify(arrHistory);
        localStorage.setItem('histories', json);
    }
    /**
     * ローカルストレージから取得した履歴JSONを配列にして返す。
     */
    function getHistoriesFromLocalStorage() {
        const json = localStorage.getItem('histories');
        const arrHistory = JSON.parse(json);
        return arrHistory;
    }
    function deleteHistory(event) {
        const index = event.currentTarget.id.replaceAll('deleteBtn', '');
        let arrHistory = getHistoriesFromLocalStorage();
        arrHistory.reverse();
        arrHistory.splice(index, 1);
        arrHistory.reverse();
        setHistoriesToLocalStorage(arrHistory);
        renderHistory();
    }


    const handleHistoryClick = (event) => {
        const history = event.currentTarget.innerText.replaceAll(' ', '').split('/');
        document.getElementById('caller-belongs').value = history[0];
        document.getElementById('caller-name').value    = history[1];
        callerBelongs = history[0];
        callerName    = history[1];
    }

    function onKeyDown(e) {
		if (e.keyCode == 13) handleMemoBtnClick();
    }
</script>

<section class="left-menu" id="left-menu">
    <div class="create-message">
        <input type="text" class="history-input" id="caller-belongs" bind:value={callerBelongs}/>
        <input type="text" class="history-input" id="caller-name" bind:value={callerName}/>
        <button class="" id="" on:click={handleMemoBtnClick} >Create Message</button>
    </div>
    <div class="history">
        <ul class="history-ul" id="history-ul">
            {#each arrHistoryBelongs as belongs, i}
                <li class="history-li" on:click={handleHistoryClick} on:keydown={handleHistoryClick}>
                    <button class="deleteBtn" id="deleteBtn{i}" on:click|stopPropagation={deleteHistory}></button>
                    <span class="history-info">{belongs} / {arrHistoryName[i]}</span>
                </li>
            {/each}
        </ul>
    </div>
</section>

<style>
    .left-menu {
        display: flex;
        flex-flow: column nowrap;
        align-items: center;
        justify-content: center;
        min-width: 310px;
        width: clamp(310px, 1rem + 15%, 500px);
        height: calc(100vh - 60px);
        color: #fff;
        background-color: #4b8fb8;
        z-index: 1;
    }
    .create-message {
        display: flex;
        flex-flow: column nowrap;
        align-items: center;
        justify-content: center;
        width: 80%;
        height: 30%;
    }
    .history-input {
        width: 100%;
    }
    .history-input:hover {
        box-shadow: 0px 0px 3px 3px rgba(255, 255, 255, .2);
    }
    .history {
        width: 80%;
        height: 70%;
        box-sizing: border-box;
        overflow-y: scroll;
        /*スクロールバー非表示（IE・Edge）*/
        -ms-overflow-style: none;
    }
    .history::-webkit-scrollbar {
        display: none;
    }
    .history-ul {
        display: flex;
        flex-flow: column nowrap;
        gap: 10px;
        width: 100%;
        list-style: none;
    }
    .history-li {
        display: flex;
        justify-content: flex-start;
        align-items: center;
        width: 100%;
        padding: 5px;
        border-radius: 4px;
        box-sizing: border-box;
        background-color: #fff;
        color: black;
        cursor: pointer;
    }
    .history-li:hover {
        color: #77cbff;
        box-shadow: 0px 0px 3px 3px rgba(255, 255, 255, .2);
    }
    .deleteBtn {
        width: 40px;
        height: 40px;
        margin: 2px;
        color: #fff;
        background-color: #dc3545;
        cursor: pointer;
        align-self: center;
    }
    .deleteBtn::after {
        content: '×';
        font-size: 2rem;
    }
    .deleteBtn:hover {
        background-color: #b82b39;
    }
    .history-info {
        padding-left: 2rem;
    }
    @media print {
        .left-menu {
            display: none;
        }
    }
   
</style>