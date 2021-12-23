<template>
    <div class="box document p-1 raises-on-hover">
        <div class="level">
            <div class="level-left">
                <div class="level-item">
                    <span class="icon is-small m-1"
                        v-tooltip="file.name">
                        <fa :icon="icon"/>
                    </span>
                    <span class="filename">{{ file.name }}</span>
                </div>
            </div>
            <div class="level-right">
                <fade>
                    <div class="level-item">
                        <a class="is-naked ml-2"
                            v-if="file.isShareable && canAccess('core.files.link')"
                            @click="link">
                            <span class="icon is-small">
                                <fa icon="link"
                                    size="sm"/>
                            </span>
                        </a>
                        <a class="is-naked ml-2"
                            @click="show"
                            v-if="file.isViewable && canAccess('core.files.show')">
                            <span class="icon is-small">
                                <fa icon="eye"
                                    size="sm"/>
                            </span>
                        </a>
                        <a class="is-naked ml-2"
                            :href="downloadLink"
                            v-if="file.isViewable && canAccess('core.files.download')">
                            <span class="icon is-small">
                                <fa icon="cloud-download-alt"
                                    size="sm"/>
                            </span>
                        </a>
                        <confirmation @confirm="$emit('delete')"
                            v-if="file.isDestroyable && canAccess('core.documents.destroy')">
                            <a class="is-naked ml-2">
                                <span class="icon is-small">
                                    <fa icon="trash-alt"
                                        size="sm"/>
                                </span>
                            </a>
                        </confirmation>
                        <dropdown :triggers="['hover']"
                            placement="top">
                            <span class="icon is-small is-naked ml-2">
                                <fa icon="info-circle"
                                    size="sm"/>
                            </span>
                            <template #popper>
                                <div class="info">
                                    <p>
                                        <span class="icon is-small">
                                            <fa icon="user"/>
                                        </span>
                                        {{ file.owner.name }}
                                    </p>
                                    <p>
                                        <span class="icon is-small">
                                            <fa icon="calendar-alt"/>
                                        </span>
                                        {{ timeFromNow(file.createdAt) }}
                                    </p>
                                    <p>
                                        <span class="icon is-small">
                                            <fa icon="database"/>
                                        </span>
                                        {{ size }} Kb
                                    </p>
                                </div>
                            </template>
                        </dropdown>
                    </div>
                </fade>
                <url :show="temporaryLink !== ''"
                    :link="temporaryLink"
                    @close="temporaryLink = ''"/>
            </div>
        </div>
    </div>
</template>

<script>
import { Dropdown } from 'v-tooltip';
import { FontAwesomeIcon as Fa } from '@fortawesome/vue-fontawesome';
import { library } from '@fortawesome/fontawesome-svg-core';
import {
    faEye, faCloudDownloadAlt, faTrashAlt, faLink,
    faInfoCircle, faUser, faCalendarAlt, faDatabase,
} from '@fortawesome/free-solid-svg-icons';
import Confirmation from '@enso-ui/confirmation/bulma';
import formatDistance from '@enso-ui/ui/src/modules/plugins/date-fns/formatDistance';
import Url from '@enso-ui/files/src/bulma/pages/files/components/Url.vue';
import { Fade } from '@enso-ui/transitions';
import { EnsoFile, numberFormat } from '@enso-ui/mixins';

library.add(
    faEye, faCloudDownloadAlt, faTrashAlt, faLink,
    faInfoCircle, faUser, faCalendarAlt, faDatabase,
);

export default {
    name: 'Document',

    components: {
        Fa, Dropdown, Confirmation, Url, Fade,
    },

    inject: ['canAccess', 'errorHandler', 'http', 'route'],

    props: {
        file: {
            type: Object,
            required: true,
        },
    },

    emits: ['delete'],

    data: () => ({
        temporaryLink: '',
    }),

    computed: {
        icon() {
            const file = new EnsoFile(this.file.name);

            return file.icon();
        },
        downloadLink() {
            return this.route('core.files.download', this.file.id);
        },
        openLink() {
            return this.route('core.files.show', this.file.id);
        },
        size() {
            return numberFormat(this.file.size);
        },
    },

    methods: {
        link() {
            this.http.get(this.route('core.files.link', this.file.id))
                .then(({ data }) => (this.temporaryLink = data.link))
                .catch(this.errorHandler);
        },
        show() {
            window.open(this.openLink, '_blank').focus();
        },
        timeFromNow(date) {
            return formatDistance(date);
        },
    },
};
</script>

<style lang="scss">
    .box.document {
        .level-left {
            flex: 1;
            min-width: 0;

            .level-item {
                flex: 1;
                min-width: 0;
                justify-content: flex-start;

                .filename {
                    white-space: nowrap;
                    overflow: hidden;
                    text-overflow: ellipsis;
                }
            }
        }
    }
</style>
