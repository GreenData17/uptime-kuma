<template>
    <div class="incident-report-container">
        <h1>{{ $t("Incident Reports") }}</h1>
        <incident-list hideViewHistoryPage :slug="this.$route.params.slug"/>
    </div>
</template>

<script>
import IncidentList from "../components/IncidentList.vue";

export default {
    components: {IncidentList},
    data() {
        return {
            slug: null,
        };
    },

    async mounted() {
        this.slug = this.$route.params.slug;
    },
};
</script>
<style>
.incident-report-container {
    padding: 32px;
}
</style>

