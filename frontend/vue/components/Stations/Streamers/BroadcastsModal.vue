<template>
    <b-modal
        id="streamer_broadcasts"
        ref="modal"
        size="lg"
        centered
        :title="$gettext('Streamer Broadcasts')"
    >
        <template v-if="listUrl">
            <div
                style="min-height: 40px;"
                class="flex-fill text-left bg-primary rounded mb-2"
            >
                <inline-player ref="player" />
            </div>

            <data-table
                id="station_streamer_broadcasts"
                ref="datatable"
                :show-toolbar="false"
                :fields="fields"
                :api-url="listUrl"
            >
                <template #cell(download)="row">
                    <template v-if="row.item.recording?.links?.download">
                        <play-button
                            class="file-icon"
                            icon-class="outlined"
                            :url="row.item.recording?.links?.download"
                        />
                        &nbsp;
                        <a
                            class="name"
                            :href="row.item.recording?.links?.download"
                            target="_blank"
                            :title="$gettext('Download')"
                        >
                            <icon icon="cloud_download" />
                        </a>
                    </template>
                    <template v-else>
&nbsp;
                    </template>
                </template>
                <template #cell(actions)="row">
                    <b-button-group size="sm">
                        <b-button
                            size="sm"
                            variant="danger"
                            @click.prevent="doDelete(row.item.links.delete)"
                        >
                            {{ $gettext('Delete') }}
                        </b-button>
                    </b-button-group>
                </template>
            </data-table>
        </template>
        <template #modal-footer>
            <b-button
                variant="default"
                @click="close"
            >
                {{ $gettext('Close') }}
            </b-button>
        </template>
    </b-modal>
</template>
<script>
import DataTable from '~/components/Common/DataTable.vue';
import formatFileSize from '~/functions/formatFileSize.js';
import InlinePlayer from '~/components/InlinePlayer';
import Icon from '~/components/Common/Icon';
import PlayButton from "~/components/Common/PlayButton";
import {DateTime} from 'luxon';
import '~/vendor/sweetalert';
import {useAzuraCast} from "~/vendor/azuracast";

export default {
    name: 'StreamerBroadcastsModal',
    components: {PlayButton, Icon, InlinePlayer, DataTable},
    data() {
        return {
            listUrl: null,
            fields: [
                {
                    key: 'download',
                    label: ' ',
                    sortable: false,
                    class: 'shrink pr-3'
                },
                {
                    key: 'timestampStart',
                    label: this.$gettext('Start Time'),
                    sortable: false,
                    formatter: (value) => {
                        const {timeConfig} = useAzuraCast();

                        return DateTime.fromSeconds(value).toLocaleString(
                            {...DateTime.DATETIME_MED, ...timeConfig}
                        );
                    },
                    class: 'pl-3'
                },
                {
                    key: 'timestampEnd',
                    label: this.$gettext('End Time'),
                    sortable: false,
                    formatter: (value) => {
                        if (value === 0) {
                            return this.$gettext('Live');
                        }

                        const {timeConfig} = useAzuraCast();

                        return DateTime.fromSeconds(value).toLocaleString(
                            {...DateTime.DATETIME_MED, ...timeConfig}
                        );
                    }
                },
                {
                    key: 'size',
                    label: this.$gettext('Size'),
                    sortable: false,
                    formatter: (value, key, item) => {
                        if (!item.recording?.size) {
                            return '';
                        }

                        return formatFileSize(item.recording.size);
                    }
                },
                {
                    key: 'actions',
                    label: this.$gettext('Actions'),
                    sortable: false,
                    class: 'shrink'
                }
            ]
        };
    },
    methods: {
        doDelete (url) {
            this.$confirmDelete({
                title: this.$gettext('Delete Broadcast?')
            }).then((result) => {
                if (result.value) {
                    this.axios.delete(url).then((resp) => {
                        this.$notifySuccess(resp.data.message);
                        this.$refs.datatable.refresh();
                    });

                    this.$refs.datatable.refresh();
                }
            });
        },
        open (listUrl) {
            this.listUrl = listUrl;
            this.$refs.modal.show();
        },
        close () {
            this.$refs.player.stop();

            this.listUrl = null;
            this.$refs.modal.hide();
        }
    }
};
</script>
