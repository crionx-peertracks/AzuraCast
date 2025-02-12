<template>
    <div class="card">
        <div class="card-header bg-primary-dark">
            <div class="d-flex align-items-center">
                <h2 class="card-title flex-fill my-0">
                    {{ $gettext('Song Playback Timeline') }}
                </h2>
                <div class="flex-shrink">
                    <a
                        id="btn-export"
                        class="btn btn-bg"
                        :href="exportUrl"
                        target="_blank"
                    >
                        <icon icon="file_download" />
                        {{ $gettext('Download CSV') }}
                    </a>

                    <date-range-dropdown
                        v-model="dateRange"
                        time-picker
                        :tz="stationTimeZone"
                        @update="relist"
                    />
                </div>
            </div>
        </div>
        <data-table
            ref="datatable"
            responsive
            paginated
            select-fields
            :fields="fields"
            :api-url="apiUrl"
        >
            <template #cell(datetime)="row">
                {{ formatTimestamp(row.item.played_at) }}
            </template>
            <template #cell(datetime_station)="row">
                {{ formatTimestampStation(row.item.played_at) }}
            </template>
            <template #cell(listeners_start)="row">
                {{ row.item.listeners_start }}
            </template>
            <template #cell(delta)="row">
                <span class="typography-subheading">
                    <template v-if="row.item.delta_total > 0">
                        <span class="text-success">
                            <icon icon="trending_up" />
                            {{ abs(row.item.delta_total) }}
                        </span>
                    </template>
                    <template v-else-if="row.item.delta_total < 0">
                        <span class="text-danger">
                            <icon icon="trending_down" />
                            {{ abs(row.item.delta_total) }}
                        </span>
                    </template>
                    <template v-else>
                        0
                    </template>
                </span>
            </template>
            <template #cell(song)="row">
                <div :class="{'text-muted': !row.item.is_visible}">
                    <template v-if="row.item.song.title">
                        <b>{{ row.item.song.title }}</b><br>
                        {{ row.item.song.artist }}
                    </template>
                    <template v-else>
                        {{ row.item.song.text }}
                    </template>
                </div>
            </template>
            <template #cell(source)="row">
                <template v-if="row.item.is_request">
                    {{ $gettext('Listener Request') }}
                </template>
                <template v-else-if="row.item.playlist">
                    {{ $gettext('Playlist:') }}
                    {{ row.item.playlist }}
                </template>
                <template v-else-if="row.item.streamer">
                    {{ $gettext('Live Streamer:') }}
                    {{ row.item.streamer }}
                </template>
                <template v-else>
                    &nbsp;
                </template>
            </template>
        </data-table>
    </div>
</template>

<script>
import Icon from "~/components/Common/Icon";
import DataTable from "~/components/Common/DataTable";
import DateRangeDropdown from "~/components/Common/DateRangeDropdown";
import {DateTime} from 'luxon';
import {useAzuraCast} from "~/vendor/azuracast";

export default {
    name: 'StationsReportsTimeline',
    components: {DateRangeDropdown, DataTable, Icon},
    props: {
        baseApiUrl: {
            type: String,
            required: true
        },
        stationTimeZone: {
            type: String,
            required: true
        }
    },
    data() {
        let nowTz = DateTime.now().setZone(this.stationTimeZone);

        return {
            dateRange: {
                startDate: nowTz.minus({days: 13}).toJSDate(),
                endDate: nowTz.toJSDate(),
            },
            fields: [
                {
                    key: 'datetime',
                    label: this.$gettext('Date/Time (Browser)'),
                    selectable: true,
                    sortable: false
                },
                {
                    key: 'datetime_station',
                    label: this.$gettext('Date/Time (Station)'),
                    sortable: false,
                    selectable: true,
                    visible: false
                },
                {
                    key: 'listeners_start',
                    label: this.$gettext('Listeners'),
                    selectable: true,
                    sortable: false
                },
                {
                    key: 'delta',
                    label: this.$gettext('Change'),
                    selectable: true,
                    sortable: false
                },
                {
                    key: 'song',
                    isRowHeader: true,
                    label: this.$gettext('Song Title'),
                    selectable: true,
                    sortable: false
                },
                {
                    key: 'source',
                    label: this.$gettext('Source'),
                    selectable: true,
                    sortable: false
                }
            ],
        }
    },
    computed: {
        apiUrl() {
            let apiUrl = new URL(this.baseApiUrl, document.location);

            let apiUrlParams = apiUrl.searchParams;
            apiUrlParams.set('start', DateTime.fromJSDate(this.dateRange.startDate).toISO());
            apiUrlParams.set('end', DateTime.fromJSDate(this.dateRange.endDate).toISO());

            return apiUrl.toString();
        },
        exportUrl() {
            let exportUrl = new URL(this.apiUrl, document.location);
            let exportUrlParams = exportUrl.searchParams;

            exportUrlParams.set('format', 'csv');

            return exportUrl.toString();
        },
    },
    methods: {
        relist() {
            this.$refs.datatable.relist();
        },
        abs(val) {
            return Math.abs(val);
        },
        formatTimestamp(unix_timestamp) {
            const {timeConfig} = useAzuraCast();

            return DateTime.fromSeconds(unix_timestamp).toLocaleString(
                {...DateTime.DATETIME_SHORT, ...timeConfig}
            );
        },
        formatTimestampStation(unix_timestamp) {
            const {timeConfig} = useAzuraCast();

            return DateTime.fromSeconds(unix_timestamp).setZone(this.stationTimeZone).toLocaleString(
                {...DateTime.DATETIME_SHORT, ...timeConfig}
            );
        }
    }
};
</script>
