<template>
    <b-card no-body>
        <b-card-header header-bg-variant="primary-dark">
            <h2 class="card-title">
                {{ $gettext('Storage Locations') }}
            </h2>
        </b-card-header>
        <b-tabs
            pills
            card
            lazy
        >
            <b-tab
                v-for="tab in tabs"
                :key="tab.type"
                :active="activeType === tab.type"
                :title="tab.title"
                no-body
                @click="setType(tab.type)"
            />
        </b-tabs>

        <b-card-body body-class="card-padding-sm">
            <b-button
                variant="outline-primary"
                @click.prevent="doCreate"
            >
                <icon icon="add" />
                {{ $gettext('Add Storage Location') }}
            </b-button>
        </b-card-body>

        <data-table
            id="admin_storage_locations"
            ref="datatable"
            :show-toolbar="false"
            :fields="fields"
            :responsive="false"
            :api-url="listUrlForType"
        >
            <template #cell(actions)="row">
                <b-button-group size="sm">
                    <b-button
                        size="sm"
                        variant="primary"
                        @click.prevent="doEdit(row.item.links.self)"
                    >
                        {{ $gettext('Edit') }}
                    </b-button>
                    <b-button
                        size="sm"
                        variant="danger"
                        @click.prevent="doDelete(row.item.links.self)"
                    >
                        {{ $gettext('Delete') }}
                    </b-button>
                </b-button-group>
            </template>
            <template #cell(adapter)="row">
                <h5 class="m-0">
                    {{ getAdapterName(row.item.adapter) }}
                </h5>
                <p class="card-text">
                    {{ row.item.uri }}
                </p>
            </template>
            <template #cell(space)="row">
                <template v-if="row.item.storageAvailable">
                    <b-progress
                        :value="row.item.storageUsedPercent"
                        show-progress
                        height="15px"
                        class="mb-1"
                        :variant="getProgressVariant(row.item.storageUsedPercent)"
                    />

                    {{ getSpaceUsed(row.item) }}
                </template>
                <template v-else>
                    {{ getSpaceUsed(row.item) }}
                </template>
            </template>
            <template #cell(stations)="row">
                {{ row.item.stations.join(', ') }}
            </template>
        </data-table>
    </b-card>

    <edit-modal
        ref="editModal"
        :create-url="listUrl"
        :type="activeType"
        @relist="relist"
    />
</template>

<script>
import DataTable from '~/components/Common/DataTable';
import EditModal from './StorageLocations/EditModal';
import Icon from '~/components/Common/Icon';

export default {
    name: 'AdminStorageLocations',
    components: {Icon, EditModal, DataTable},
    props: {
        listUrl: {
            type: String,
            required: true
        }
    },
    data() {
        return {
            activeType: 'station_media',
            fields: [
                {key: 'adapter', label: this.$gettext('Adapter'), sortable: false},
                {key: 'space', label: this.$gettext('Space Used'), class: 'text-nowrap', sortable: false},
                {key: 'stations', label: this.$gettext('Station(s)'), sortable: false},
                {key: 'actions', label: this.$gettext('Actions'), class: 'shrink', sortable: false}
            ]
        };
    },
    computed: {
        tabs() {
            return [
                {
                    type: 'station_media',
                    title: this.$gettext('Station Media')
                },
                {
                    type: 'station_recordings',
                    title: this.$gettext('Station Recordings')
                },
                {
                    type: 'station_podcasts',
                    title: this.$gettext('Station Podcasts'),
                },
                {
                    type: 'backup',
                    title: this.$gettext('Backups')
                }
            ]
        },
        listUrlForType() {
            return this.listUrl + '?type=' + this.activeType;
        }
    },
    methods: {
        setType (type) {
            this.activeType = type;
            this.relist();
        },
        getAdapterName (adapter) {
            switch (adapter) {
                case 'local':
                    return this.$gettext('Local');

                case 's3':
                    return this.$gettext('Remote: S3 Compatible');

                case 'dropbox':
                    return this.$gettext('Remote: Dropbox');
                    
                case 'sftp':
                    return this.$gettext('Remote: SFTP');
            }
        },
        getSpaceUsed(item) {
            return (item.storageAvailable)
                ? item.storageUsed + ' / ' + item.storageAvailable
                : item.storageUsed;
        },
        getProgressVariant(percent) {
            if (percent > 85) {
                return 'danger';
            } else if (percent > 65) {
                return 'warning';
            } else {
                return 'default';
            }
        },
        relist() {
            this.$refs.datatable.refresh();
        },
        doCreate() {
            this.$refs.editModal.create();
        },
        doEdit(url) {
            this.$refs.editModal.edit(url);
        },
        doModify(url) {
            this.$notify(this.$gettext('Applying changes...'), {
                variant: 'warning'
            });

            this.axios.put(url).then((resp) => {
                this.$notifySuccess(resp.data.message);
                this.relist();
            });
        },
        doDelete (url) {
            this.$confirmDelete({
                title: this.$gettext('Delete Storage Location?'),
            }).then((result) => {
                if (result.value) {
                    this.$wrapWithLoading(
                        this.axios.delete(url)
                    ).then((resp) => {
                        this.$notifySuccess(resp.data.message);
                        this.relist();
                    });
                }
            });
        }
    }
};
</script>
