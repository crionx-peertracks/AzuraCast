<template>
    <b-card no-body>
        <b-card-header header-bg-variant="primary-dark">
            <h2 class="card-title">
                {{ $gettext('Stations') }}
            </h2>
        </b-card-header>

        <b-card-body body-class="card-padding-sm">
            <b-button
                variant="outline-primary"
                @click.prevent="doCreate"
            >
                <icon icon="add" />
                {{ $gettext('Add Station') }}
            </b-button>
        </b-card-body>

        <data-table
            id="stations"
            ref="datatable"
            paginated
            :fields="fields"
            :api-url="listUrl"
        >
            <template #cell(name)="row">
                <div class="typography-subheading">
                    {{ row.item.name }}
                </div>
                <code>{{ row.item.short_name }}</code>
            </template>
            <template #cell(frontend_type)="row">
                {{ getFrontendName(row.item.frontend_type) }}
            </template>
            <template #cell(backend_type)="row">
                {{ getBackendName(row.item.backend_type) }}
            </template>
            <template #cell(actions)="row">
                <b-button-group size="sm">
                    <b-button
                        size="sm"
                        variant="secondary"
                        :href="row.item.links.manage"
                        target="_blank"
                    >
                        {{ $gettext('Manage') }}
                    </b-button>
                    <b-button
                        size="sm"
                        variant="secondary"
                        @click.prevent="doClone(row.item.name, row.item.links.clone)"
                    >
                        {{ $gettext('Clone') }}
                    </b-button>
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
        </data-table>
    </b-card>

    <admin-stations-edit-modal
        v-bind="$props"
        ref="editModal"
        :create-url="listUrl"
        @relist="relist"
    />

    <admin-stations-clone-modal
        ref="cloneModal"
        @relist="relist"
    />
</template>

<script>
import DataTable from '~/components/Common/DataTable';
import Icon from '~/components/Common/Icon';
import AdminStationsEditModal from "./Stations/EditModal";
import {get} from "lodash";
import AdminStationsCloneModal from "./Stations/CloneModal";
import stationFormProps from "./Stations/stationFormProps";

export default {
    name: 'AdminPermissions',
    components: {AdminStationsCloneModal, AdminStationsEditModal, Icon, DataTable},
    props: {
        ...stationFormProps,
        listUrl: {
            type: String,
            required: true
        },
        frontendTypes: {
            type: Object,
            required: true
        },
        backendTypes: {
            type: Object,
            required: true
        }
    },
    data() {
        return {
            fields: [
                {key: 'name', isRowHeader: true, label: this.$gettext('Name'), sortable: true},
                {key: 'frontend_type', label: this.$gettext('Broadcasting'), sortable: false},
                {key: 'backend_type', label: this.$gettext('AutoDJ'), sortable: false},
                {key: 'actions', label: this.$gettext('Actions'), sortable: false, class: 'shrink'}
            ]
        };
    },
    methods: {
        relist() {
            this.$refs.datatable.refresh();
        },
        doCreate() {
            this.$refs.editModal.create();
        },
        doEdit(url) {
            this.$refs.editModal.edit(url);
        },
        doClone(stationName, url) {
            this.$refs.cloneModal.create(stationName, url);
        },
        doDelete(url) {
            this.$confirmDelete({
                title: this.$gettext('Delete Station?'),
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
        },
        getFrontendName(frontend_type) {
            return get(this.frontendTypes, [frontend_type, 'name'], '');
        },
        getBackendName(backend_type) {
            return get(this.backendTypes, [backend_type, 'name'], '');
        }
    }
};
</script>
