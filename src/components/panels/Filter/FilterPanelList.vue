<template>
    <responsive
        :breakpoints="{
            mobile: (el) => el.width <= 395,
        }">
        <template #default="{ el }">
            <v-simple-table class="temperature-panel-table">
                <thead>
                    <tr>
                        <th class="icon">&nbsp;</th>
                        <th class="name">{{ $t('Panels.FilterPanel.Name') }}</th>
                        <th v-if="!el.is.mobile" class="state">
                            {{ $t('Panels.FilterPanel.State') }}
                        </th>
                        <th class="current">
                            {{ $t('Panels.FilterPanel.Current') }}
                        </th>
                    </tr>
                </thead>
                <tbody>
                    <filter-panel-list-item-nevermore
                        v-if="existsNevermoreFilter"
                        :is-responsive-mobile="el.is.mobile ?? false" />
                    <filter-panel-list-item
                        v-for="objectName in temperature_sensors"
                        :key="objectName"
                        :object-name="objectName"
                        :is-responsive-mobile="el.is.mobile ?? false" />
                </tbody>
            </v-simple-table>
        </template>
    </responsive>
</template>

<script lang="ts">
import Component from 'vue-class-component'
import { Mixins } from 'vue-property-decorator'
import BaseMixin from '@/components/mixins/base'
import TemperaturePanelListItemNevermore from '@/components/panels/Temperature/TemperaturePanelListItemNevermore.vue'
import FilterPanelListItemNevermore from '@/components/panels/Filter/FilterPanelListItemNevermore.vue'

@Component({
    components: { FilterPanelListItemNevermore, TemperaturePanelListItemNevermore },
})
export default class TemperaturePanelList extends Mixins(BaseMixin) {
    get available_heaters() {
        return this.$store.state.printer?.heaters?.available_heaters ?? []
    }

    get filteredHeaters() {
        return this.available_heaters
            .filter((fullName: string) => {
                const splits = fullName.split(' ')
                let name = splits[0]
                if (splits.length > 1) name = splits[1]

                return !name.startsWith('_')
            })
            .sort(this.sortObjectName)
    }

    get available_sensors() {
        return this.$store.state.printer?.heaters?.available_sensors ?? []
    }

    get available_monitors() {
        return this.$store.state.printer?.heaters?.available_monitors ?? []
    }

    get monitors() {
        return this.available_monitors.sort(this.sortObjectName)
    }

    get temperature_fans() {
        return this.available_sensors
            .filter((name: string) => name.startsWith('temperature_fan') && !name.startsWith('temperature_fan _'))
            .sort(this.sortObjectName)
    }

    get existsNevermoreFilter() {
        return 'nevermore' in this.$store.state.printer
    }

    get hideMcuHostSensors(): boolean {
        return this.$store.state.gui.view.tempchart.hideMcuHostSensors ?? false
    }

    get hideMonitors(): boolean {
        return this.$store.state.gui.view.tempchart.hideMonitors ?? false
    }

    get temperature_sensors() {
        let sensors = this.available_sensors.filter((fullName: string) => {
            if (this.available_heaters.includes(fullName)) return false
            if (this.temperature_fans.includes(fullName)) return false
            if (!fullName.includes('VOC')) return false

            // hide MCU & Host sensors, if the function is enabled
            if (this.hideMcuHostSensors && this.checkMcuHostSensor(fullName)) return false

            const splits = fullName.split(' ')
            let name = splits[0]
            if (splits.length > 1) name = splits[1]

            return !name.startsWith('_')
        })
        return sensors
    }

    get heaterObjects() {
        return [...this.filteredHeaters, ...this.temperature_fans]
    }

    get settings() {
        return this.$store.state.printer?.configfile?.settings ?? {}
    }

    checkMcuHostSensor(fullName: string) {
        const settingsObject = this.settings[fullName.toLowerCase()] ?? {}
        const sensor_type = settingsObject.sensor_type ?? ''

        return ['temperature_mcu', 'temperature_host'].includes(sensor_type)
    }

    sortObjectName(a: string, b: string) {
        const splitsA = a.split(' ')
        let nameA = splitsA[0]
        if (splitsA.length > 1) nameA = splitsA[1]
        nameA = nameA.toUpperCase()

        const splitsB = b.split(' ')
        let nameB = splitsB[0]
        if (splitsB.length > 1) nameB = splitsB[1]
        nameB = nameB.toUpperCase()

        if (nameA < nameB) return -1
        if (nameA > nameB) return 1

        return 0
    }
}
</script>

<style scoped>
.temperature-panel-table th,
.temperature-panel-table ::v-deep td {
    padding-top: 5px !important;
    padding-bottom: 5px !important;
    height: auto !important;
}

.temperature-panel-table ::v-deep tr:hover {
    background: none !important;
}

.temperature-panel-table ::v-deep .icon {
    width: 24px;
    padding-right: 0 !important;
    text-align: center;
}

.temperature-panel-table ::v-deep .name {
    width: 100px;
    text-align: left !important;
}

.temperature-panel-table ::v-deep .state {
    width: 100px;
    text-align: right !important;
}

.temperature-panel-table ::v-deep .current {
    width: 100px;
    text-align: center !important;
}
</style>
