<template>
    <b-card no-body>
        <b-card-header header-bg-variant="primary-dark">
            <h2 class="card-title">
                {{ $gettext('Roles & Permissions') }}
            </h2>
        </b-card-header>

        <info-card>
            <p class="card-text">
                {{
                    $gettext('AzuraCast uses a role-based access control system. Roles are given permissions to certain sections of the site, then users are assigned into those roles.')
                }}
            </p>
        </info-card>

        <b-card-body body-class="card-padding-sm">
            <b-button
                variant="outline-primary"
                @click.prevent="doCreate"
            >
                <icon icon="add" />
                {{ $gettext('Add Role') }}
            </b-button>
        </b-card-body>

        <data-table
            id="permissions"
            ref="datatable"
            paginated
            :fields="fields"
            :api-url="listUrl"
        >
            <template #cell(permissions)="row">
                <div v-if="row.item.permissions.global.length > 0">
                    {{ $gettext('Global') }}
                    :
                    {{ getGlobalPermissionNames(row.item.permissions.global).join(', ') }}
                </div>
                <div
                    v-for="(permissions, stationId) in row.item.permissions.station"
                    :key="stationId"
                >
                    <b>{{ getStationName(stationId) }}</b>:
                    {{ getStationPermissionNames(permissions).join(', ') }}
                </div>
            </template>
            <template #cell(actions)="row">
                <b-button-group
                    v-if="!row.item.is_super_admin"
                    size="sm"
                >
                    <b-button
                        size="sm"
                        variant="primary"
                        @click.prevent="doEdit(row.item.links.self)"
                    >
                        {{ $gettext('Edit') }}
                    </b-button>
                    <b-button
                        v-if="row.item.id !== 1"
                        size="sm"
                        variant="danger"
                        @click.prevent="doDelete(row.item.links.self)"
                    >
                        {{ $gettext('Delete') }}
                    </b-button>
                </b-button-group>
            </template>
        </data-table>
    </b-card>

    <edit-modal
        ref="editModal"
        :create-url="listUrl"
        :station-permissions="stationPermissions"
        :stations="stations"
        :global-permissions="globalPermissions"
        @relist="relist"
    />
</template>

<script>
import DataTable from '~/components/Common/DataTable';
import EditModal from './Permissions/EditModal';
import Icon from '~/components/Common/Icon';
import InfoCard from '~/components/Common/InfoCard';
import {filter, get, map} from 'lodash';

export default {
    name: 'AdminPermissions',
    components: {InfoCard, Icon, EditModal, DataTable},
    props: {
        listUrl: {
            type: String,
            required: true
        },
        stations: {
            type: Array,
            required: true
        },
        globalPermissions: {
            type: Array,
            required: true
        },
        stationPermissions: {
            type: Array,
            required: true
        }
    },
    data() {
        return {
            fields: [
                {key: 'name', isRowHeader: true, label: this.$gettext('Role Name'), sortable: true},
                {key: 'permissions', label: this.$gettext('Permissions'), sortable: false},
                {key: 'actions', label: this.$gettext('Actions'), sortable: false, class: 'shrink'}
            ]
        };
    },
    methods: {
        getGlobalPermissionNames(permissions) {
            return filter(map(permissions, (permission) => {
                return get(this.globalPermissions, permission, null);
            }));
        },
        getStationPermissionNames(permissions) {
            return filter(map(permissions, (permission) => {
                return get(this.stationPermissions, permission, null);
            }));
        },
        getStationName(stationId) {
            return get(this.stations, stationId, null);
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
        doDelete(url) {
            this.$confirmDelete({
                title: this.$gettext('Delete Role?'),
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
