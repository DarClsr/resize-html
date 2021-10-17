<template>
    <div class="sizeBox">
        <resize-item />
        <resize-div/>
    </div>
</template>

<script>
import ResizeItem from "@/components/resize-item/index.vue"
import ResizeDiv from "@/components/resize-div/index.vue"
export default {
    components:{
        ResizeItem,
        ResizeDiv
    }
}
</script>