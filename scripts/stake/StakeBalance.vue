<script setup>
import { computed, defineEmits, ref, toRefs, watch, nextTick } from 'vue';
import { optimiseCurrencyLocale } from '../global.js';
import { translation, ALERTS } from '../i18n.js';
import Modal from '../Modal.vue';
import { createAlert, isColdAddress } from '../misc';
import { COIN, cChainParams } from '../chain_params';
import { beautifyNumber } from '../misc';

const coldStakingAddress = defineModel('coldStakingAddress');
const csAddrInternal = ref(coldStakingAddress.value);
watch(coldStakingAddress, (addr) => (csAddrInternal.value = addr));
const showColdStakingAddressModal = ref(false);
const emit = defineEmits(['showUnstake', 'showStake', 'setColdStakingAddress']);
const coldAddrInput = ref(null);
const props = defineProps({
    coldBalance: Number,
    price: Number,
    currency: String,
    displayDecimals: Number,
});
const { coldBalance, price, currency, displayDecimals } = toRefs(props);
const coldBalanceStr = computed(() => {
    const nCoins = coldBalance.value / COIN;
    const strBal = nCoins.toFixed(displayDecimals.value);
    const nLen = strBal.length;
    return beautifyNumber(strBal, nLen >= 10 ? '17px' : '25px');
});
const coldBalanceValue = computed(() => {
    const { nValue, cLocale } = optimiseCurrencyLocale(
        (coldBalance.value / COIN) * price.value
    );

    return nValue.toLocaleString('en-gb', cLocale);
});

watch(showColdStakingAddressModal, (showColdStakingAddressModal) => {
    nextTick(() => {
        if (showColdStakingAddressModal) coldAddrInput.value.focus();
    });
});

function submit() {
    if (csAddrInternal.value === '') {
        csAddrInternal.value = cChainParams.current.defaultColdStakingAddress;
    }
    if (isColdAddress(csAddrInternal.value)) {
        coldStakingAddress.value = csAddrInternal.value;
        showColdStakingAddressModal.value = false;
        createAlert('info', ALERTS.STAKE_ADDR_SET, 5000);
    } else {
        createAlert('warning', ALERTS.STAKE_ADDR_BAD, 2500);
    }
}
</script>

<template>
    <div class="dcWallet-balances mb-4" style="margin-top: 37px">
        <div class="row lessBot p-0">
            <div
                class="col-6 d-flex dcWallet-topLeftMenu"
                style="justify-content: flex-start"
            >
                <h3 class="noselect balance-title"></h3>
            </div>

            <div
                class="col-6 d-flex dcWallet-topRightMenu"
                style="justify-content: flex-end"
            >
                <div class="btn-group dropleft">
                    <i
                        class="fa-solid fa-gear"
                        style="width: 20px"
                        data-testid="setColdStakeButton"
                        @click="showColdStakingAddressModal = true"
                    ></i>
                </div>
            </div>
        </div>

        <span data-i18n="staking">Staking</span><br />
        <span
            class="dcWallet-pivxBalance"
            data-testid="coldBalance"
            v-html="coldBalanceStr"
        ></span>
        <span class="dcWallet-pivxTicker">&nbsp;PIV&nbsp;</span><br />
        <div class="dcWallet-usdBalance">
            <span class="dcWallet-usdValue" data-testid="coldBalanceValue">
                {{ coldBalanceValue }}
            </span>
            <span class="dcWallet-usdValue" data-testid="coldBalanceCurrency"
                >&nbsp;{{ currency }}&nbsp;</span
            >
        </div>

        <div class="row lessTop p-0">
            <div class="col-6 d-flex" style="justify-content: flex-start">
                <div
                    data-i18n="stake"
                    class="dcWallet-btn-left"
                    data-testid="showStakeButton"
                    @click="emit('showStake')"
                >
                    Stake
                </div>
            </div>

            <div class="col-6 d-flex" style="justify-content: flex-end">
                <div
                    data-i18n="stakeUnstake"
                    class="dcWallet-btn-right"
                    data-testid="showUnstakeButton"
                    @click="emit('showUnstake')"
                >
                    Unstake
                </div>
            </div>
        </div>
    </div>
    <Teleport to="body">
        <Modal :show="showColdStakingAddressModal">
            <template #header>
                <h3
                    class="modal-title"
                    style="text-align: center; width: 100%; color: #8e21ff"
                >
                    Set your Cold Staking address
                </h3>
            </template>
            <template #body>
                <p>
                    <span
                        style="opacity: 0.65; margin: 10px; margin-buttom: 0px"
                    >
                        {{ translation.popupColdStakeNote }}
                    </span>
                </p>
                <input
                    type="text"
                    ref="coldAddrInput"
                    :placeholder="`${
                        translation.popupExample
                    } ${coldStakingAddress.substring(0, 6)}...`"
                    data-testid="csAddrInput"
                    v-model="csAddrInternal"
                    style="text-align: center"
                />
            </template>
            <template #footer>
                <button
                    type="button"
                    class="pivx-button-big"
                    style="float: right"
                    data-testid="csAddrSubmit"
                    @click="submit()"
                >
                    {{ translation.popupConfirm }}
                </button>
                <button
                    type="button"
                    class="pivx-button-big"
                    style="float: right; opacity: 0.7"
                    data-testid="csAddrCancel"
                    @click="showColdStakingAddressModal = false"
                >
                    {{ translation.popupCancel }}
                </button>
            </template>
        </Modal>
    </Teleport>
</template>
