<template>
    <section
        class="card"
        role="region"
    >
        <div class="card-header bg-primary-dark">
            <h2 class="card-title">
                {{ $gettext('Custom Fallback File') }}
            </h2>
        </div>

        <info-card>
            <p class="card-text">
                {{ $gettext('This file will be played on your radio station any time no media is scheduled to play or a critical error occurs that interrupts regular broadcasting.') }}
            </p>
        </info-card>

        <div class="card-body">
            <b-form-group>
                <div class="form-row">
                    <b-form-group
                        class="col-md-6"
                        label-for="intro_file"
                    >
                        <template #label>
                            {{ $gettext('Select Custom Fallback File') }}
                        </template>

                        <flow-upload
                            :target-url="apiUrl"
                            :valid-mime-types="['audio/*']"
                            @success="onFileSuccess"
                        />
                    </b-form-group>

                    <b-form-group class="col-md-6">
                        <template #label>
                            {{ $gettext('Current Custom Fallback File') }}
                        </template>

                        <div v-if="hasFallback">
                            <div class="buttons pt-3">
                                <b-button
                                    block
                                    variant="bg"
                                    :href="apiUrl"
                                    target="_blank"
                                >
                                    {{ $gettext('Download') }}
                                </b-button>
                                <b-button
                                    block
                                    variant="danger"
                                    @click="deleteFallback"
                                >
                                    {{ $gettext('Clear File') }}
                                </b-button>
                            </div>
                        </div>
                        <div v-else>
                            {{ $gettext('There is no existing custom fallback file associated with this station.') }}
                        </div>
                    </b-form-group>
                </div>
            </b-form-group>
        </div>
    </section>
</template>

<script setup>
import FlowUpload from '~/components/Common/FlowUpload';
import InfoCard from "~/components/Common/InfoCard";
import {ref} from "vue";
import {useAxios} from "~/vendor/axios";

const props = defineProps({
    apiUrl: {
        type: String,
        required: true
    },
    recordHasFallback: {
        type: Boolean,
        required: true
    }
});

const hasFallback = ref(props.recordHasFallback);

const onFileSuccess = () => {
    hasFallback.value = true;
};

const {axios} = useAxios();

const deleteFallback = () => {
    axios.delete(props.apiUrl).then(() => {
        hasFallback.value = false;
    });
}
</script>
