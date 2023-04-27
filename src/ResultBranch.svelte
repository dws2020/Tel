
<script>
    import { branchList } from './branchList.js';
    export let branchIndex;
</script>

<ul>
{#each branchList[branchIndex].branches as branch}
    <li>{branch.branchName}</li>
{/each}
</ul>

<style>

</style>