<script setup>
import { useAlerts } from '../composables/use_alerts';
import { computed, watch, ref } from 'vue';
import Alert from './Alert.vue';

const alerts = useAlerts();
const foldedAlerts = ref([]);
watch(alerts, () => {
    const res = [];
    let previousAlert;
    let count = 1;
    const pushAlert = () => {
        if (previousAlert) {
            const countStr = count === 1 ? '' : ` (x${count})`;
            const timeout =
                previousAlert.created + previousAlert.timeout - Date.now();
            const show = timeout > 0;
            if (!show) return;
            const alert = ref({
                ...previousAlert,
                message: `${previousAlert.message}${countStr}`,
                show,
                // Store original message so we can use it as key.
                // This skips the animation in case of multiple errors
                original: previousAlert.message,
            });

            res.push(alert);
            if (timeout > 0) {
                setTimeout(() => {
                    alert.value.show = false;
                }, timeout);
            }
        }
    };
    for (const alert of alerts.alerts) {
        if (previousAlert && previousAlert?.message === alert.message) {
            count++;
        } else {
            pushAlert();
            count = 1;
        }
        previousAlert = alert;
    }
    pushAlert();
    foldedAlerts.value = res;
});
</script>

<template>
    <transition-group name="alert">
        <div
            v-for="alert of foldedAlerts.filter((a) => a.value.show)"
            :key="alert.value.original"
        >
            <Alert
                :message="alert.value.message"
                :level="alert.value.level"
                @click="alert.value.show = false"
            />
        </div>
    </transition-group>
</template>

<style>
.alert-enter-active,
.alert-leave-active {
    transition: all 0.5s ease;
}
.alert-enter-from,
.alert-leave-to {
    opacity: 0;
}
</style>
