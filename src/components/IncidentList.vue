<template>
    <a v-if="incidentReports.length !== 0 && hideViewHistoryPage === false" :href="slug + '/incidents'">View all incidents</a>

    <div v-if="incidentReports.length" class="incident-history">
        <div v-for="incident in incidentReports" :key="incident.id" class="incident-item">
            <div class="incident-timeline">
                <div class="incident-timeline-icon">
                    <font-awesome-icon :icon="incidentIcon[incident.style]" :class="incident.style"></font-awesome-icon>
                </div>
            </div>

            <div class="incident-card">
                <div class="incident-header">
                    <b>{{ incident.title }}</b>
                    <div class="incident-header-datetime">
                        <span>{{ $t("Date Created") }}: {{ $root.datetime(incident.createdDate) }} ({{ dateFromNow(incident.createdDate) }})</span>
                        <br />
                        <span v-if="incident.lastUpdatedDate">{{ $t("Last Updated") }}: {{ $root.datetime(incident.lastUpdatedDate) }} ({{ dateFromNow(incident.lastUpdatedDate) }})</span>
                    </div>
                </div>
                <p>{{ incident.content }}</p>
            </div>
        </div>

        <div class="incident-timeline-end">
        </div>
    </div>
    <p v-else>{{ $t("NoIncidentOrError") }}</p>
</template>

<script>
import axios from "axios";
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";
import dayjs from "dayjs";

export default {
    name: "IncidentList",
    components: { FontAwesomeIcon },

    props: {
        editMode: {
            type: Boolean,
        },
        slug: {
            type: String,
            required: true
        },
        hideViewHistoryPage: {
            type: Boolean,
        },
    },

    data() {
        return {
            expanded: [],
            incidentReports: [],
            isLoading: false,
            error: null,
            incidentIcon: {
                primary: "check-circle",
                warning: "exclamation-circle",
                danger: "exclamation-circle",
                info: "info-circle",
                light: "info-circle",
                dark: "info-circle"
            }
        };
    },

    watch: {
        incidents: {
            handler(newVal) {
                if (newVal && newVal.length) {
                    console.log("Incidents ready:", newVal);
                }
            },
            immediate: true
        }
    },

    mounted() {
        this.fetchIncidentReports();
    },

    methods: {
        /**
         * Get the relative time difference of a date from now
         * @param {any} date Date to get time difference
         * @returns {string} Time difference
         */
        dateFromNow(date) {
            return dayjs.utc(date).fromNow();
        },

        /**
         * A quick and dirty way to convert the database naming convention
         * into javascript naming convention
         * notice: might want to add this to the util file?
         * @param {any} obj any kind of js object from the database
         * @returns {{}|*} converted js object with js naming convention
         */
        snakeToCamel(obj) {
            if (Array.isArray(obj)) {
                return obj.map(v => this.snakeToCamel(v));
            } else if (obj !== null && obj.constructor === Object) {
                return Object.keys(obj).reduce((acc, key) => {
                    const camelKey = key.replace(/_([a-z])/g, (_, c) => c.toUpperCase());
                    acc[camelKey] = this.snakeToCamel(obj[key]);
                    return acc;
                }, {});
            }
            return obj;
        },

        /**
         * Fetches the incident reports
         * Credit to @MasonColoretti
         * @returns {Promise<void>} result of the fetching request
         */
        async fetchIncidentReports() {
            this.isLoading = true;

            try {
                if (this.editMode) {
                    // WebSocket mode
                    const socket = this.$root.getSocket?.();
                    if (!socket) {
                        throw new Error("Socket not initialized");
                    }

                    socket.emit("getStatusPageIncidentHistory", this.slug, (data) => {
                        if (data.ok) {
                            this.incidentReports = data.incidents;
                        } else {
                            this.error = data.msg;
                            console.error("Error fetching incident reports:", data.msg);
                        }
                        this.isLoading = false;
                    });
                } else {
                    // REST API fallback
                    const res = await axios.get(`/api/status-page/${this.slug}/incidents`);
                    this.incidentReports = this.snakeToCamel(res.data.incidents) || [];
                    this.isLoading = false;
                }
            } catch (error) {
                this.error = error;
                console.error("Error fetching incident reports:", error);
                this.isLoading = false;
            }
        }
    },
};
</script>

<style lang="scss" scoped>
@import "../assets/vars.scss";

.incident-history {
    display: flex;
    flex-flow: column;
    padding: 10px 0;

    .incident-item {
        display: flex;

        &:first-child .incident-timeline {
            border-radius: 10px 10px 0 0;
        }

        &:last-child .incident-timeline {
            border-radius: 0 0 10px 10px;
        }

        .incident-timeline {
            height: auto;
            width: 4px;

            background-color: #f3f3f3;

            body.dark & {
                background-color: #191f29;
            }

            .incident-timeline-icon {
                position: relative;
                top: calc(50% - 14px);
                left: calc(50% - 14px);
                font-size: 24px;
                line-height: 22px;
                border-radius: 50%;
                border: solid 2px #e4e4e4;

                background-color: #e4e4e4;
                width: 28px;
                height: 28px;

                body.dark & {
                    border: solid 2px #444444;
                    background-color: #444444;
                }

                .primary {
                    color: $primary;
                }

                .warning {
                    color: $warning;
                }

                .danger {
                    color: $danger;
                }

                .info {
                    color: var(--bs-info);
                }

                .light {
                    color: #ffffff;
                }

                .dark {
                    color: #444444;
                    background-color: #e4e4e4;
                    border-radius: 50%;
                }
            }
        }

        .incident-card {
            width: 100%;

            box-shadow: 0 15px 70px rgba(0, 0, 0, 0.1);
            border-radius: 10px;
            padding: 10px;
            margin: 10px 10px 10px 24px;

            body.dark & {
                background-color: #0d1117;
            }

            .incident-header {
                display: flex;
                align-items: stretch;
                justify-content: space-between;
                padding-bottom: 10px;

                .incident-header-datetime {
                    font-size: 12px;
                }
            }

            p {
                word-break: break-all;
            }
        }
    }

    .incident-timeline-end {
        height: 50px;
        width: 4px;

        border-left: 4px #f3f3f3 dashed;

        body.dark & {
            border-left-color: #191f29;
        }
    }
}

</style>
