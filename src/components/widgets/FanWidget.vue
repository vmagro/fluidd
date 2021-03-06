<template>
  <div>
    <input-slider
      value-suffix="%"
      input-xs
      v-model.number="value"
      :label="fan.prettyName"
      :rules="rules"
      :disabled="!klippyConnected"
      :readonly="!fan.controllable">
    </input-slider>
    <v-divider class="my-1" v-if="divider"></v-divider>
  </div>
</template>

<script lang="ts">
import { Component, Mixins, Prop } from 'vue-property-decorator'
import InputSlider from '@/components/inputs/InputSlider.vue'
import UtilsMixin from '@/mixins/utils'
import { Waits } from '@/globals'
import { Fan } from '@/store/socket/types'

@Component({
  components: {
    InputSlider
  }
})
export default class FanWidget extends Mixins(UtilsMixin) {
  @Prop({ type: Object, required: true })
  fan!: Fan

  @Prop({ type: Boolean, default: false })
  divider!: boolean

  get value () {
    return (this.fan.speed) ? this.fan.speed * 100 : 0
  }

  set value (target: number) {
    // If this is a controllable fan, it's either the part fan [fan] or a generic fan [fan_generic].
    if (this.fan.type === 'fan') {
      target = Math.ceil(target * 2.55)
      this.sendGcode(`M106 S${target}`, Waits.onSetFanSpeed)
    }
    if (this.fan.type === 'fan_generic') {
      target = target / 100
      this.sendGcode(`SET_FAN_SPEED FAN=${this.fan.name} SPEED=${target}`, Waits.onSetFanSpeed)
    }
  }

  get offBelow () {
    const config = this.$store.getters['socket/getPrinterSettings'](this.fan.name) || {}
    const offBelow = config.off_below || 0
    return offBelow * 100
  }

  rules = [
    (v: string) => {
      if (this.offBelow <= 0) return true
      return (parseInt(v) >= this.offBelow || parseInt(v) === 0) || 'min error'
    }
  ]
}
</script>
