<template>
    <section
        class="card"
        role="region"
    >
        <div class="card-header bg-primary-dark">
            <h2 class="card-title">
                {{ $gettext('Upload Stereo Tool Configuration') }}
            </h2>
        </div>

        <info-card>
            <p class="card-text">
                {{ $gettext('Stereo Tool is an industry standard for software audio processing. For more information on how to configure it, please refer to the') }}
                <a
                    href="https://www.thimeo.com/stereo-tool/"
                    target="_blank"
                >
                    {{ $gettext('Stereo Tool documentation.') }}
                </a>
            </p>
        </info-card>

        <div class="card-body">
            <b-form-group>
                <div class="form-row">
                    <b-form-group
                        class="col-md-6"
                        label-for="stereo_tool_configuration_file"
                    >
                        <template #label>
                            {{ $gettext('Select Configuration File') }}
                        </template>
                        <template #description>
                            {{
                                $gettext('This configuration file should be a valid .sts file exported from Stereo Tool.')
                            }}
                        </template>

                        <flow-upload
                            :target-url="apiUrl"
                            :valid-mime-types="['text/plain']"
                            @success="onFileSuccess"
                        />
                    </b-form-group>

                    <b-form-group class="col-md-6">
                        <template #label>
                            {{ $gettext('Current Configuration File') }}
                        </template>
                        <div v-if="hasStereoToolConfiguration">
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
                                    @click="deleteConfigurationFile"
                                >
                                    {{ $gettext('Clear File') }}
                                </b-button>
                            </div>
                        </div>
                        <div v-else>
                            {{ $gettext('There is no Stereo Tool configuration file present.') }}
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
import {mayNeedRestartProps, useMayNeedRestart} from "~/functions/useMayNeedRestart";
import {useNotify} from "~/vendor/bootstrapVue";
import {useAxios} from "~/vendor/axios";

const props = defineProps({
    ...mayNeedRestartProps,
    recordHasStereoToolConfiguration: {
        type: Boolean,
        required: true
    },
    apiUrl: {
        type: String,
        required: true
    }
});

const hasStereoToolConfiguration = ref(props.recordHasStereoToolConfiguration);

const {mayNeedRestart} = useMayNeedRestart(props.restartStatusUrl);

const onFileSuccess = () => {
    mayNeedRestart();
    hasStereoToolConfiguration.value = true;
};

const {wrapWithLoading, notifySuccess} = useNotify();
const {axios} = useAxios();

const deleteConfigurationFile = () => {
    wrapWithLoading(
        axios({
            method: 'DELETE',
            url: this.apiUrl
        })
    ).then(() => {
        mayNeedRestart();
        hasStereoToolConfiguration.value = false;
        notifySuccess();
    });
};
</script>
