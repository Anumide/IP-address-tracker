<template>
    <div class="px-6">
        <span class="text-slate-500 text-[10px] font-semibold tracking-wider mb-3"><small>{{ subTitle }}</small></span>
        <p class="text-black font-semibold">{{ title }}</p>
    </div>
</template>
 
<script>
 export default{ 
 name: 'ipInfo',
 props: ['subTitle', 'title']
}
</script>
 
<style scoped>
 
</style>