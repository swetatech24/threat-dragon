<template>
    <div class="td-report">
        <b-row class="no-print td-report-options sticky">
            <b-col>
                <b-form class="">
                    <b-form-row>
                        <b-col>
                            <b-form-group
                                label-cols="auto"
                                id="outofscope-group">
                                <b-form-checkbox
                                    id="show_outofscope"
                                    v-model="display.outOfScope"
                                >{{ $t('report.options.showOutOfScope') }}</b-form-checkbox>
                            </b-form-group>
                        </b-col>

                        <b-col>
                            <b-form-group
                                label-cols="auto"
                                id="mitigated-group">
                                <b-form-checkbox
                                    id="show_mitigated"
                                    v-model="display.mitigated"
                                >{{ $t('report.options.showMitigatedThreats') }}</b-form-checkbox>
                            </b-form-group>
                        </b-col>

                        <b-col>
                            <b-form-group
                                label-cols="auto"
                                id="model-group">
                                <b-form-checkbox
                                    id="show_models"
                                    v-model="display.diagrams"
                                >{{ $t('report.options.showModelDiagrams') }}</b-form-checkbox>
                            </b-form-group>
                        </b-col>

                        <b-col>
                            <b-form-group
                                label-cols="auto"
                                id="branding-group">
                                <b-form-checkbox
                                    id="show_branding"
                                    v-model="display.branding"
                                >{{ $t('report.options.showBranding') }}</b-form-checkbox>
                            </b-form-group>
                        </b-col>
                    </b-form-row>
                </b-form>
            </b-col>

            <b-col class="text-right right">
                <b-btn-group>
                    <td-form-button
                        id="td-print-btn"
                        :onBtnClick="print"
                        icon="print"
                        :text="$t('forms.print')" />
                    <td-form-button
                        id="td-return-btn"
                        :isPrimary="true"
                        :onBtnClick="onCloseClick"
                        icon="times"
                        :text="$t('forms.close')" />
                </b-btn-group>
            </b-col>
        </b-row>

        <div v-if="!!model" class="td-report-container">
            <div class="td-report-section">
                <td-coversheet :branding="display.branding" />
                <td-print-coversheet
                    :title="model.summary.title"
                    :owner="model.summary.owner"
                    :reviewer="model.detail.reviewer"
                    :contributors="contributors"
                    :branding="display.branding"
                ></td-print-coversheet>
            </div>

            <div class="td-report-section">
                <td-executive-summary
                    :summary="model.summary.description"
                    :threats="allThreats"
                ></td-executive-summary>
                <td-print-executive-summary
                    :summary="model.summary.description"
                    :threats="allThreats"
                ></td-print-executive-summary>
            </div>

            <td-diagram-detail
                v-for="(diagram, idx) in model.detail.diagrams"
                :key="idx"
                :diagram="diagram"
                :showOutOfScope="display.outOfScope"
                :showMitigated="display.mitigated"
                :showDiagram="display.diagrams"
            ></td-diagram-detail>
        </div>
    </div>
</template>

<style lang="scss">
.td-report-options label {
    padding-top: 4px;
    font-size: 12px !important;
}

.card-header {
    font-size: 16px;
}
</style>

<style lang="scss" scoped>

.td-branding {
    padding-left: 50px;
}

.td-report {
    font-size: 12px;
}

.td-report-section {
    margin-top: 15px;
}

.td-report-container {
    margin-top: 5px;
}

.sticky {
    position: sticky;
    top: 45px;
    width: 100%;
    background-color: $white;
    padding-top: 15px;
    z-index: 9999;
}

.right {
    right: 0;
}
</style>

<script>
import { mapState, mapGetters } from 'vuex';

import { getProviderType } from '@/service/provider/providers.js';
import TdCoversheet from '@/components/report/Coversheet.vue';
import TdDiagramDetail from '@/components/report/DiagramDetail.vue';
import TdExecutiveSummary from '@/components/report/ExecutiveSummary.vue';
import TdFormButton from '@/components/FormButton.vue';
import TdPrintCoversheet from '@/components/printed-report/Coversheet.vue';
import TdPrintExecutiveSummary from '@/components/printed-report/ExecutiveSummary.vue';
import threatService from '@/service/threats/index.js';

export default {
    name: 'ReportModel',
    components: {
        TdCoversheet,
        TdDiagramDetail,
        TdExecutiveSummary,
        TdFormButton,
        TdPrintCoversheet,
        TdPrintExecutiveSummary
    },
    data() {
        return {
            display: {
                outOfScope: true,
                mitigated: true,
                diagrams: true,
                branding: true
            }
        };
    },
    computed: {
        ...mapState({
            model: (state) => state.threatmodel.data,
            providerType: (state) => getProviderType(state.provider.selected),
            allThreats: function (state) {
                return threatService.filter(state.threatmodel.data.detail.diagrams, {
                    showOutOfScope: true,
                    showMitigated: true
                });
            }
        }),
        ...mapGetters({
            contributors: 'contributors'
        })
    },
    methods: {
        onCloseClick() {
            this.$router.push({ name: `${this.providerType}ThreatModel`, params: this.$route.params });
        },
        print() {
            window.print();
        }
    }
};

</script>
