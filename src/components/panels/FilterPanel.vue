<template>
    <panel
        v-if="klipperReadyForGui"
        :icon="mdiThermometerLines"
        :title="$t('Panels.FilterPanel.Headline')"
        :collapsible="true"
        card-class="filter-panel">
        <v-card-text class="pa-0">
            <filter-panel-list />
            <template v-if="boolTempchart">
                <v-divider class="my-0" />
                <filter-chart />
            </template>
        </v-card-text>
    </panel>
</template>

<script lang="ts">
import Component from 'vue-class-component'
import { Mixins } from 'vue-property-decorator'
import { capitalize, convertName } from '@/plugins/helpers'
import BaseMixin from '@/components/mixins/base'
import ControlMixin from '@/components/mixins/control'
import TempChart from '@/components/charts/TempChart.vue'
import TemperatureInput from '@/components/inputs/TemperatureInput.vue'
import Panel from '@/components/ui/Panel.vue'
import Responsive from '@/components/ui/Responsive.vue'
import { mdiCloseThick, mdiThermometerLines } from '@mdi/js'
import TemperaturePanelPresets from '@/components/panels/Temperature/TemperaturePanelPresets.vue'
import FilterChart from '@/components/charts/FilterChart.vue'

@Component({
    components: { FilterChart, Panel, TempChart, TemperatureInput, Responsive, TemperaturePanelPresets },
})
export default class FilterPanel extends Mixins(BaseMixin, ControlMixin) {
    mdiCloseThick = mdiCloseThick
    mdiThermometerLines = mdiThermometerLines

    convertName = convertName
    capitalize = capitalize

    get boolTempchart(): boolean {
        return this.$store.state.gui.view.tempchart.boolTempchart ?? false
    }
}
</script>
