<script>
    export let callerInfo;
    export let state;
    export let callTime;
    import { createEventDispatcher } from 'svelte';
    const dispatch = createEventDispatcher();

    // コンポーネント共通変数
    const myMember = [
        'A部長',
        'Bさん',
        'Cさん',
        'Dさん',
        'イーサン',
        '　'
    ];
    let callTarget = "";

    // 伝言メモ作成
    const selectTarget = (event) => {
        callTarget = event.target.innerText;
        clearChecked();
        event.target.classList.add('checked');
    }
    function clearChecked() {
        let myMemberLi = document.querySelectorAll('.my_member_li');
        myMemberLi.forEach(ele => {
            ele.classList.remove('checked');
        });
    }
    

    const goPrint = (event) => {
        if (callTarget == '') {
            alert('相手が指定されていません。');
            return;
        }
        let presetMessage = event.target.innerText;
        let startOfMemo;
        switch (presetMessage) {
            case '折り返し電話依頼':
                startOfMemo = '折り返し電話をいただきたいとの伝言がありました。';
                break;
            case '電話があった旨の伝言':
                startOfMemo = '電話があった旨の伝言がありました。';
                break;
            case '下記の伝言':
                startOfMemo = '下記の伝言がありました。';
            default:
        }
        let messageContent   = document.getElementById('message_inner');
        messageContent.value = startOfMemo + '\n' +messageContent.value;
        print();
        dispatch('generatePrint');
        // 伝言内容を消す処理は架電者を選択した時の処理に入れた方がいい？
        clearMessage();
        clearChecked();
    }
    function clearMessage() {
        callTarget = '';
        document.getElementById('message_inner').value = '';
    }

    const void0 = () => {
        return;
    }
</script>

<section class="right-menu { state }">

    <section class="my_member">
        <ol class="my_member_ol">
            {#each myMember as member}
                <li class="my_member_li" on:click|stopPropagation={selectTarget} on:keydown={void0}>{member}</li>
            {/each}
        </ol>
    </section>
    <section class="detail">
        <div class="call-time">{callTime}</div>
        <div class="call-target"><span class="strong">{callTarget}</span> 様へ</div>
        <div class="caller-belongs"><span class="strong">{callerInfo.belongs}</span> の</div>
        <div class="caller-kana">{callerInfo.kana}</div>
        <div class="caller-name"><span class="strong">{callerInfo.name}</span> 様より電話がありました。</div>
        <div class="buttons">
            <button class="preset-btn" on:click={goPrint}>折り返し電話依頼</button>
            <button class="preset-btn" on:click={goPrint}>電話があった旨の伝言</button>
            <button class="preset-btn" on:click={goPrint}>下記の伝言</button>
        </div>
    </section>
    <section class="message">
        <textarea class="message_inner" id="message_inner"></textarea>
        <p class="signature">受付者：自分の名前</p>
    </section>
</section>

<style>
    .right-menu {
        display: grid;
        grid-template:
        "member  detail " 70%
        "message message" 30% 
        / 40% 60%;
		background-color: #fff;
		width: 601px;
		height: 100vh;
		position: absolute;
		top: 0;
		right: -601px;
		z-index: 999;
		transition: all .3s cubic-bezier(.16, .41, .68, .91);
		box-shadow: 0 -2px 15px 1px rgba(0, 0, 0, .3);
	}
	.right-menu.show {
		right: 0px;
	}

    /* 自部署メンバー */
    .my_member {
        grid-area: member;
        display: flex;
        justify-content: center;
        overflow: scroll;
    }
    .my_member_ol {
        width: 90%;
        counter-reset:list;
        list-style-type:none;
        font: 14px/1.6 'arial narrow', sans-serif;
        padding:0;
    }
    .my_member_li {
        cursor: pointer;
        position:relative;
        margin: 7px 0 7px 0px;
        padding-left:40px;
        font-weight: bold;
        font-size:14px;
        line-height: 30px;
        border: solid 1px #F6A38B;
        border-radius:20px;
        -webkit-transition: 0.3s;
        -moz-transition: 0.3s;
        -o-transition: 0.3s;
        -ms-transition: 0.3s;
        transition: 0.3s;
    }
    .my_member_li::before {
        counter-increment: list;
        content: counter(list);
        position: absolute;
        left: 0px;
        width: 30px;
        height: 30px;
        text-align: center;
        color: #fff;
        line-height:30px;
        background: #F6A38B;
        border-radius: 50%;
        top: 50%;
        -moz-transform: translateY(-50%);
        -webkit-transform: translateY(-50%);
        -o-transform: translateY(-50%);
        -ms-transform: translateY(-50%);
        transform: translateY(-50%);
    }
    .my_member_li:hover,
    .my_member_li.checked {
        background: #F6A38B;
        color: #fff;
    }
    .my_member_li:hover::before {
        background: #fff;
        color: #F6A38B;
    }

    /* 電話情報 */
    .detail {
        grid-area: detail;
        padding: 10px;
    }
    .detail > div {
        margin-bottom: 10px;
    }
    .strong {
        font-size: 1.6rem;
        border-bottom: 1px solid #601;
        padding: 0 10px;
    }
    .call-target {
        width: 100%;
    }
    .call-time {
        width: 100%;
    }
    .caller-belongs {
        width: 100%;
    }
    .detail .caller-kana {
        height: 11px;
        width: 100%;
        font-size: 1.3rem;
        font-weight: lighter;
        padding-left: 10px;
        margin-bottom: 0;
    }
    .caller-name {
        width: 100%;
    }

    /* 対応方法ボタン */
    .buttons {
        display: flex;
        flex-flow: column nowrap;
        margin-top: 20px;
    }
    .preset-btn {
        cursor: pointer;
        width: 100%;
        margin-bottom: 10px !important;
        display: flex;
        justify-content: center;
        align-items: center;
        margin:0 auto;
        padding: 0.7rem 2rem;
        border: 1px solid #2589d0;
        border-radius: 5px;
        background-color: #fff;
        color: #2589d0;
    }
    .preset-btn:hover {
        background-color: #2589d0;
        color: #fff;
    }

    /* 伝言メモ欄 */
    .message {
        display: flex;
        justify-content: center;
        align-items: center;
        grid-area: message;
        width: 100%;
    }
    .message_inner {
        width: 95%;
        height: 90%;
    }
    .signature {
        display: none;
    }

    /* 印刷用CSS */
    @media print {
        .right-menu {
            display: flex;
            flex-flow: column nowrap;
            align-items: center;
            margin: 0 auto;
            width: 200mm;
            box-shadow: none;
        }
        .my_member, .buttons {
            display: none;
            width: 0mm;
        }
        .detail {
            display: block;
            width: 200mm;
        }
        .message {
            display: block;
            width: 200mm;
            height: 100mm;
        }
        .signature {
            display: block;
            text-align: right;
            width: 95%;
        } 
    }
</style>