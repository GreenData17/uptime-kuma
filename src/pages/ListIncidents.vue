<template>
    <div>
        <h1>{{ $t("Incident Reports") }}</h1>
        <div v-if="isLoading">Loading...</div>
        <div v-else-if="filteredReports.length">
            <div
                v-for="report in filteredReports"
                :key="report.id"
                class="big-padding"
            >
                <h3>{{ datetimeFormat(report.createdDate) }}</h3>
                <hr />
                <h4>{{ report.title }}</h4>
                <p>{{ report.content }}</p>
                <hr />
                <br /><br />
            </div>
        </div>
        <p v-else>No incident reports found or an error occurred.</p>
    </div>
</template>

<script>
import axios from "axios";
import dayjs from "dayjs";

export default {
    data() {
        return {
            slug: null,
            incidentReports: [],
            isLoading: false,
            error: null,
        };
    },
    computed: {
        filteredReports() {
            return this.incidentReports
                .slice() // Create a copy to avoid mutating the original array
                .sort(
                    (a, b) =>
                        new Date(b.created_date) - new Date(a.created_date),
                )
                .slice(-25); // Get the last 25 sorted reports
        },
    },

    mounted() {
        this.slug = this.$route.params.slug;
        this.fetchIncidentReports();
    },
    methods: {
        async fetchIncidentReports() {
            this.isLoading = true;
            try {
                const res = await axios.get(`/api/status-page/${this.slug}/incidents`);
                this.incidentReports = res.data.incidents || [];
            } catch (error) {
                this.error = error;
                console.error("Error fetching incident reports:", error);
            } finally {
                this.isLoading = false;
            }
        },

        /**
         * Return a value in a custom format
         * @param {any} value Value to format
         * @param {any} format Format to return value in
         * @returns {string} Formatted string
         */
        datetimeFormat(value, format) {
            if (value !== undefined && value !== "") {
                return dayjs.utc(value).tz(this.timezone).format(format);
            }
            return "";
        },
    },
};
</script>
<style>
.incident-report-container {
    display: flex;
    flex-direction: column;
    gap: 10px; /* Adjust gap between boxes */
}

.incident-report {
    background-color: #fff;
    border-radius: 10px;
    padding: 20px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

</style>

