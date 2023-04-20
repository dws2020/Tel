<script>
	// issue
	// タブで右メニューにフォーカスがいく問題
	// メインエリアのエリア表示
	// 自分の所属を指定してマイメンバーを変更
    import Header from "./Header.svelte";
    import LeftMenu from "./LeftMenu.svelte";
	import EmpList from './EmpList.svelte';
	import RightMenu from './RightMenu.svelte';
	import { branchList } from './branchList.js';
    import ResultBranch from "./ResultBranch.svelte";

	// コンポーネント共通変数
	let callerBelongs;
	let displayNone;
	let callerInfo = {};
	let state;
	let callTime;
	let branchID = 0;

	const handleListClick = (event) => {
		callerBelongs = event.detail.targetBranch;
		displayNone = 'display-none';
	}
	const handleEmpListClick = (event) => {
		rightMenuShow();
		let eventChildren   = event.detail.targetEmp.children;
		callerInfo.name     = eventChildren[2].innerText;
		callerInfo.empNo    = eventChildren[3].innerText;
		callerInfo.kana     = eventChildren[4].innerText;
		callerInfo.belongs  = eventChildren[5].innerText;
		callerInfo.position = eventChildren[6].innerText;
	}

	const handleLogoClick = () => {
		callerBelongs = '';
		displayNone   = '';
	}
	const rightMenuShow = () => {
		const wDays = ['日', '月', '火', '水', '木', '金', '土']
		state       = "show";
		let timeNow = new Date();
		let year    = timeNow.getFullYear();
		let month   = timeNow.getMonth() + 1;
		let day     = timeNow.getDate();
		let wDay    = wDays[timeNow.getDay()];
		let minute  = String(timeNow.getMinutes()).length === 1 ? '0' + timeNow.getMinutes() : timeNow.getMinutes();
		let time    = timeNow.getHours() + ':' + minute;
		callTime = `${year}/${month}/${day}(${wDay}) ${time}`;
	}
	const closeRightMenu = () => {
		state = "";
	}
	function onKeyDown(e) {
		// state = e.keyCode == 27 ? '' : state;
	}

	// mainセクションのエリアをホバーしたとき
    const showBranches = (event) => {
        branchID = event.target.id;
    }

</script>

<!-- ヘッダー -->
<Header on:listClick={handleListClick} on:logoClick={handleLogoClick}/>

<!-- メインレイアウト（メイン・両サイドメニュー） -->
<div class="container">
	<LeftMenu on:createMemo={handleEmpListClick} />
	<main>
		<section class="greeting {displayNone}">
			<h1>Hello World</h1>
			<p>The world is your oyster.</p>
			<button on:click = { rightMenuShow }>slide open</button>
			<!-- <nav class="main-menu">
				<ul class="area-list">
					{#each branchList as branchObj, i}
					<li>
						<p id={i} on:mouseover={showBranches} on:focus={showBranches}>{ branchObj.branchArea }</p>
					</li>
					{/each}
				</ul>
				<ResultBranch branchIndex={branchID}/>
			</nav> -->
		</section>
		<EmpList belongs={callerBelongs} on:empListClick={handleEmpListClick} />
	</main>
	<div class="dark-layer { state }" on:click={ closeRightMenu } on:keydown={ onKeyDown }></div>
	<RightMenu state={state} callTime={callTime} callerInfo={callerInfo} on:generatePrint={closeRightMenu} />
</div>

<style>
	/* 全体 ---------------------------------------- */
	.container {
		display: flex;
		width: 100vw;
		overflow: hidden;
		background-color: #f9f9f9;
	}

	/* メイン ---------------------------------------- */
	main {
		width: 100%;
		text-align: center;
		padding: 0 1em 1em 1em;
		margin: 0 auto;
	}
	.greeting.display-none {
		display: none;
	}
	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}
	.main-menu {
		display: flex;

	}
	.main-menu ul {
		display: flex;
		flex-flow: row wrap;
        list-style: none;
    }
    .area-list {
        overflow: scroll;
        width: 100%;
        will-change: transform;
    }
    .area-list::-webkit-scrollbar {
        display: none;
    }

	/* 右メニュー ---------------------------------------- */
	.dark-layer {
		display:  none;
		width: 100vw;
		height: 100vh;
		position: absolute;
		top: 0;
		left: 0;
		background-color: rgba(0, 0, 0, .5);
		transition: all .3s cubic-bezier(.16,.41,.68,.91);
		z-index: 998;
	}
	.dark-layer.show {
		display: block;
	}
	.right-menu {
		background-color: #fff;
		width: 601px;
		height: 100vh;
		position: absolute;
		top: 0;
		right: -601px;
		z-index: 999;
		transition: all .3s cubic-bezier(.16,.41,.68,.91);
		box-shadow: 0 -2px 15px 1px rgba(0, 0, 0, .3);
		
	}
	.right-menu.show {
		right: 0px;
	}



	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}

</style>