<template>
    <v-row class="card-container" v-on:click.prevent @click="openInsurance">
        <v-col cols="12" align-self="start">
            <v-row class="d-flex flex-row align-center header-row" justify="center" :style="{'--card-background': getBgColor()}">
                <span class="currency ml-5">
                    <v-img src="/assets/currencies/insurance/insurancePolygon.svg"/>
                </span>
                <v-row class="d-flex flex-column align-start mr-3 ml-8">
                    <v-row class="d-flex" align="center">
                        <label class="card-title">
                          USD+ Insurance
                        </label>
                    </v-row>
                    <v-row class="d-flex" align="center">
                        <label class="card-title">
                          ON polygon
                        </label>
                        <div>
                            <v-img :src="require('@/assets/icon/betaGray.svg')" />
                        </div>
                    </v-row>
                    <v-row class="d-flex mt-4">
                        <label class="percentage">
                            {{ avgApy ? ((avgApy.apy > 500 ? '>500' : $utils.formatMoneyComma(avgApy.apy, 0)) + '%') : '—' }}
                        </label>
                        <label class="apy ml-3">
                          30-DAY APY
                        </label>
                    </v-row>
                </v-row>
            </v-row>

            <v-container class="mt-6">
                <v-row class="ma-0 mb-8 box">
                    <v-col cols="12">

                        <v-row class="ma-0 d-flex justify-space-between  info-container">
                          <label class="card-info-label mt-2 card-about-label ">
                            OVERALL 30-DAY APY CONSISTS OF:
                          </label>
                           <div class="tooltip-info">
                            <Tooltip :size="$wu.isFull() ? 18 : ($wu.isTablet() ? 16 : 14)"
                              :maxWidth="300"
                              text="Overall APY is the result of compounding (not addition) of APYs coming from insurance premiums and reserve investment."/>
                            </div>
                        </v-row>

                      <v-row class="ma-0 mt-3 insurance-card-info-row d-flex justify-space-between info-container">
                          <label class="card-info-label mt-2">
                            Insurance premiums
                          </label>
                          <label class="card-info-value mt-2 value-info">
                            {{ avgApy ? ((avgApy.apyPremium > 500 ? '>500' : $utils.formatMoneyComma(avgApy.apyPremium, 0)) + '%') : '—' }}
                          </label>
                          <div class="tooltip-info">
                            <Tooltip :size="$wu.isFull() ? 18 : ($wu.isTablet() ? 16 : 14)"
                                     :maxWidth="300"
                                     text="Retained portion of the USD+ yield"/>
                          </div>
                        </v-row>

                        <v-row class="ma-0 mt-3 insurance-card-info-row d-flex justify-space-between info-container">
                          <label class="card-info-label mt-2">
                            Reserve investment
                          </label>
                          <label class="card-info-value mt-2 value-info">
                            {{ avgApy ? ((avgApy.apyInvestment > 500 ? '>500' : $utils.formatMoneyComma(avgApy.apyInvestment, 0)) + '%') : '—' }}
                          </label>
                          <div class="tooltip-info">
                            <Tooltip :size="$wu.isFull() ? 18 : ($wu.isTablet() ? 16 : 14)"
                                     text="Yield generated by investing <a>insurance reserves.</a>"
                                     link="https://app.overnight.fi/insurance/polygon"/>
                          </div>
                        </v-row>
                    </v-col>
                </v-row>

                <v-row class="ma-0 insurance-card-info-row d-flex justify-space-between">
                    <label class="card-info-label mt-2">Insurance fund</label>
                    <label class="card-info-value mt-2">${{ $utils.formatMoneyComma(totalInsuranceSupply.polygon, 2) }}</label>
                </v-row>

                <v-row class="ma-0 mt-3 insurance-card-info-row d-flex justify-space-between">
                    <label class="card-info-label mt-2">Insurance coverage</label>
                    <label class="card-info-value mt-2">{{ (coverageValue !== null) ? ($utils.formatMoneyComma(coverageValue, 2) + '%') : '—' }}</label>
                </v-row>

                <v-row class="ma-0 mt-3 insurance-card-info-row d-flex justify-space-between">
                    <label class="card-info-label mt-2">Risk factor</label>
                    <label class="card-info-value mt-2">VERY HIGH</label>
                </v-row>
            </v-container>
        </v-col>

        <v-col align-self="end">
            <v-container :class="$wu.isFull() ? 'mb-6' : 'mb-6'">
                <v-row class="d-flex ma-0" align="center">
                    <label class="your-deposit">
                      Your deposit
                    </label>
                    <v-spacer></v-spacer>
                    <label class="your-deposit" :class="dataHidden ? 'hidden-label' : ''">
                        {{ dataHidden ? '' : (this.insuranceBalance.polygon && this.insuranceBalance.polygon > 0) ? ('$' + $utils.formatMoneyComma(this.insuranceBalance.polygon, 2)) : "—"  }}
                    </label>
                    <label class="your-deposit ml-1">
                        <v-icon color="var(--disabled-value)">
                            {{ insuranceRedemptionData.request === 'CAN_WITHDRAW' ? 'mdi-lock-open-variant' : 'mdi-lock' }}
                        </v-icon>
                    </label>
                </v-row>

                <v-row class="d-flex justify-space-between ma-0 mt-2">
                    <template v-if="networkId === 137">
                        <v-col :class="insuranceRedemptionData.request === 'CAN_WITHDRAW' ? 'mr-1' : ''">
                            <v-row>
                                <v-btn class="button btn-filled" @click.stop="mintAction">Mint</v-btn>
                            </v-row>
                        </v-col>
                        <v-col class="ml-1" v-if="insuranceRedemptionData.request === 'CAN_WITHDRAW'">
                            <v-row>
                                <v-btn class="button btn-outlined" @click.stop="redeemAction" outlined>
                                    REDEEM
                                </v-btn>
                            </v-row>
                        </v-col>
                    </template>

                    <template v-else>
                        <v-col>
                            <v-row>
                                <v-btn class="button btn-outlined" @click.stop="setWalletNetwork('137')" outlined>
                                    SWITCH TO POLYGON TO MINT
                                </v-btn>
                            </v-row>
                        </v-col>
                    </template>
                </v-row>
            </v-container>

            <v-row class="footer-row d-flex align-center justify-center" @click.stop="openInsurance">
                <label class="footer-link">View USD+ Insurance performance</label>
                <img class="open-icon ml-1" src="@/assets/icon/open-in-new.svg">
            </v-row>
        </v-col>

        <resize-observer @notify="$forceUpdate()"/>
    </v-row>
</template>

<script>
import Tooltip from "@/components/common/element/Tooltip";
import {mapActions, mapGetters, mapMutations} from "vuex";
import { insuranceApiService } from "@/services/insurance-api-service";

export default {
    name: "InsuranceCard",

    components: {
        Tooltip
    },

    props: {},

    computed: {
        ...mapGetters("accountData", ["insuranceBalance"]),
        ...mapGetters("supplyData", ["totalInsuranceSupply"]),
        ...mapGetters("statsData", ["totalUsdPlusValue"]),
        ...mapGetters("insuranceData", ["insuranceStrategyData", 'insuranceRedemptionData']),
        ...mapGetters("network", ["appApiUrl", "networkId", "networkName", "polygonConfig"]),
        ...mapGetters('magicEye', ['dataHidden']),

        coverageValue: function () {
            if (this.totalInsuranceSupply && this.totalUsdPlusValue) {
                return 100.00 * this.totalInsuranceSupply.polygon / this.totalUsdPlusValue;
            } else {
                return null;
            }
        },
    },

     data() {
        return {
            avgApy: null,
            isLoaded: false,
        }
    },

    watch: {
    },

    mounted() {
        this.loadAvgApyInfo()
    },

    methods: {
        ...mapActions("swapModal", ["showSwapModal", "showMintView"]),
        ...mapActions("swapModal", ["showSwapModal", "showRedeemView"]),
        ...mapActions("insuranceInvestModal", ["showInvestModal", "showMintView", "showRedeemView"]),
        ...mapActions("network", ["setWalletNetwork"]),
        ...mapActions("insuranceData", ["refreshInsurance"]),


        loadAvgApyInfo() {
            this.isLoaded = false;
            insuranceApiService.getAvgApyInfo(this.polygonConfig.appApiUrl, 'month')
            .then(data => {
                console.log('Load avg apy info for insurance card', data);
                this.avgApy = {
                    apy: data.value,
                    apyInvestment: data.apyInvestment,
                    apyPremium: data.apyPremium
                }
                this.isLoaded = true;
            })
            .catch(e => {
                console.error('Error load avg apy info for insurance card', e);
                this.isLoaded = true;
            })

        },

        mintAction() {
            this.showMintView();
            this.showInvestModal();
        },

        redeemAction() {
            this.showRedeemView();
            this.showInvestModal();
        },

        openInsurance() {
            this.goToAction('/insurance/polygon');
        },

        goToAction(id) {
            this.$router.push(id);
        },

        getBgColor() {
            return 'radial-gradient(108.67% 595.92% at 100% -3.25%, #001845 0%, #001845 27.05%, #0C255B 52.07%, #7E46E3 100%)';
        },
    }
};
</script>

<style scoped>

/* mobile */
@media only screen and (max-width: 960px) {
    .button {
        height: 36px !important;

        font-style: normal !important;
        font-weight: 400 !important;
        font-size: 16px !important;
        line-height: 20px !important;
        letter-spacing: 0.02em !important;
    }

    .currency {
        height: 60px !important;
        width: 60px !important;
    }

    .your-deposit {
        font-style: normal;
        font-weight: 700;
        font-size: 14px;
        line-height: 22px;
    }

    .footer-link {
        font-style: normal;
        font-weight: 400;
        font-size: 12px;
        line-height: 16px;
    }

    .header-row {
        height: 100px !important;
    }

    .card-container {
    }

    .card-title {
        font-style: normal;
        font-weight: 600;
        font-size: 14px;
        line-height: 16px;
        letter-spacing: 0.04em;
    }

    .percentage {
        font-style: normal;
        font-weight: 600;
        font-size: 40px;
        line-height: 40px;
    }

    .apy {
        font-style: normal;
        font-weight: 600;
        font-size: 14px;
        line-height: 16px;
    }

    .tooltip {
        margin-top: -4px;
    }

    .apy-box-total-label {
        font-style: normal;
        font-weight: 400;
        font-size: 14px;
        line-height: 18px;
        letter-spacing: 0.03em;
    }

    .apy-box-subtitle-label {
        font-style: normal;
        font-weight: 400;
        font-size: 11px;
        line-height: 16px;
        letter-spacing: 0.03em;
    }

    .card-info-label {
        font-style: normal;
        font-weight: 300;
        font-size: 16px;
        line-height: 24px;
    }

    .card-info-value {
        font-style: normal;
        font-weight: 400;
        font-size: 16px;
        line-height: 24px;
    }
}

/* tablet */
@media only screen and (min-width: 960px) and (max-width: 1400px) {
    .button {
        height: 36px !important;

        font-style: normal !important;
        font-weight: 400 !important;
        font-size: 16px !important;
        line-height: 20px !important;
        letter-spacing: 0.02em !important;
    }

    .currency {
        height: 60px !important;
        width: 60px !important;
    }

    .your-deposit {
        font-style: normal;
        font-weight: 700;
        font-size: 14px;
        line-height: 22px;
    }

    .footer-link {
        font-style: normal;
        font-weight: 400;
        font-size: 14px;
        line-height: 22px;
    }

    .header-row {
        height: 110px !important;
    }

    .card-container {
        max-width: 440px;
        max-height: 600px;
    }

    .card-title {
        font-style: normal;
        font-weight: 600;
        font-size: 16px;
        line-height: 20px;
        letter-spacing: 0.04em;
    }

    .percentage {
        font-style: normal;
        font-weight: 600;
        font-size: 50px;
        line-height: 56px;
    }

    .apy {
        font-style: normal;
        font-weight: 600;
        font-size: 16px;
        line-height: 22px;
    }

    .tooltip {
        margin-top: 1px;
    }

    .apy-box-total-label {
        font-style: normal;
        font-weight: 400;
        font-size: 16px;
        line-height: 20px;
        letter-spacing: 0.02em;
    }

    .apy-box-subtitle-label {
        font-style: normal;
        font-weight: 400;
        font-size: 12px;
        line-height: 16px;
        letter-spacing: 0.03em;
    }

    .card-info-label {
        font-style: normal;
        font-weight: 300;
        font-size: 16px;
        line-height: 24px;
    }

    .card-info-value {
        font-style: normal;
        font-weight: 400;
        font-size: 16px;
        line-height: 24px;
    }

    .hidden-label {
        width: 130px;
        height: 28px;
        background: var(--hide-account);
    }
}

/* full */
@media only screen and (min-width: 1400px) {
    .button {
        height: 36px !important;

        font-style: normal !important;
        font-weight: 400 !important;
        font-size: 16px !important;
        line-height: 20px !important;
        letter-spacing: 0.02em !important;
    }

    .currency {
        height: 78px !important;
        width: 78px !important;
    }

    .your-deposit {
        font-style: normal;
        font-weight: 800;
        font-size: 16px;
        line-height: 24px;
        letter-spacing: 0.05em;
    }

    .footer-link {
        font-style: normal;
        font-weight: 400;
        font-size: 14px;
        line-height: 22px;
    }

    .header-row {
        max-width: 460px;
        height: 130px !important;
    }

    .card-container {
        max-width: 460px;
    }

    .card-title {
        font-style: normal;
        font-weight: 600;
        font-size: 16px;
        line-height: 18px;
        letter-spacing: 0.04em;
    }

    .percentage {
        font-style: normal;
        font-weight: 600;
        font-size: 50px;
        line-height: 48px;
    }

    .apy {
        font-style: normal;
        font-weight: 600;
        font-size: 20px;
        line-height: 24px;
    }

    .tooltip {
        margin-top: 2px;
    }

    .apy-box-total-label {
        font-style: normal;
        font-weight: 400;
        font-size: 16px;
        line-height: 20px;
        letter-spacing: 0.02em;
    }

    .apy-box-subtitle-label {
        font-style: normal;
        font-weight: 400;
        font-size: 12px;
        line-height: 16px;
        letter-spacing: 0.03em;
    }

    .card-info-label {
        font-style: normal;
        font-weight: 300;
        font-size: 18px;
        line-height: 28px;
    }

    .card-info-value {
        font-style: normal;
        font-weight: 400;
        font-size: 18px;
        line-height: 28px;
    }

    .hidden-label {
        width: 130px;
        height: 28px;
        background: var(--hide-account);
    }
}

@media
only screen and (-webkit-min-device-pixel-ratio: 2)      and (min-width: 1300px),
only screen and (   min--moz-device-pixel-ratio: 2)      and (min-width: 1300px),
only screen and (     -o-min-device-pixel-ratio: 2/1)    and (min-width: 1300px),
only screen and (        min-device-pixel-ratio: 2)      and (min-width: 1300px),
only screen and (                min-resolution: 192dpi) and (min-width: 1300px),
only screen and (                min-resolution: 2dppx)  and (min-width: 1300px) {
    .card-title {
        font-style: normal;
        font-weight: 600;
        font-size: 16px;
        line-height: 18px;
        letter-spacing: 0.03em;
    }

    .percentage {
        font-style: normal;
        font-weight: 600;
        font-size: 38px;
        line-height: 42px;
    }

    .apy {
        font-style: normal;
        font-weight: 600;
        font-size: 18px;
        line-height: 24px;
    }

    .button {
        height: 36px !important;

        font-style: normal !important;
        font-weight: 400 !important;
        font-size: 15px !important;
        line-height: 20px !important;
        letter-spacing: 0.02em !important;
    }

    .currency {
        height: 78px !important;
        width: 78px !important;
    }

    .your-deposit {
        font-style: normal;
        font-weight: 800;
        font-size: 15px;
        line-height: 24px;
        letter-spacing: 0.05em;
    }

    .footer-link {
        font-style: normal;
        font-weight: 400;
        font-size: 14px;
        line-height: 20px;
    }

    .header-row {
        max-width: 460px;
        height: 130px !important;
    }

    .card-container {
        max-width: 460px;
    }

    .tooltip {
        margin-top: 1px;
    }

    .apy-box-total-label {
        font-style: normal;
        font-weight: 400;
        font-size: 16px;
        line-height: 20px;
        letter-spacing: 0.02em;
    }

    .apy-box-subtitle-label {
        font-style: normal;
        font-weight: 400;
        font-size: 12px;
        line-height: 16px;
        letter-spacing: 0.03em;
    }

    .card-info-label {
        font-style: normal;
        font-weight: 300;
        font-size: 16px;
        line-height: 24px;
    }

    .card-info-value {
        font-style: normal;
        font-weight: 400;
        font-size: 16px;
        line-height: 24px;
    }
}

.card-container {
    background: var(--secondary) !important;
    border-radius: 15px !important;
    height: 100% !important;
    filter: drop-shadow(0px 4px 4px rgba(0, 0, 0, 0.25));
}

.header-row {
    border-radius: 15px 15px 0 0;
    background: var(--card-background);
}

.card-title {
    font-family: 'Roboto', sans-serif;
    text-transform: uppercase;
    font-feature-settings: 'pnum' on, 'lnum' on;
    color: #FFFFFF;
}

.percentage {
    font-family: 'Roboto', sans-serif;
    font-feature-settings: 'pnum' on, 'lnum' on;
    color: #FFFFFF;
}

.apy {
    font-family: 'Roboto', sans-serif;
    font-feature-settings: 'pnum' on, 'lnum' on;
    color: #FFFFFF;
}

.box {
    background-color: var(--scroll-color);
    border-radius: 4px;
    text-align: center;
    width: 100%;
}

.your-deposit {
    font-family: 'Roboto', sans-serif;
    font-style: normal;
    font-feature-settings: 'liga' off;
}

.your-deposit {
    color: var(--main-gray-text);
}

.button {
    border-radius: 2px;
    box-shadow: none !important;
    width: 100% !important;

    font-family: 'Roboto', sans-serif !important;
    text-align: center !important;
    text-transform: uppercase !important;
    font-feature-settings: 'pnum' on, 'lnum' on !important;
}

.btn-filled {
    background: var(--blue-gradient);
    color: #FFFFFF !important;
}

.btn-outlined {
    color: var(--links-blue) !important;
}

.footer-row {
    background-color: var(--scroll-color) !important;
    padding: 5px;
    border-radius: 0 0 15px 15px;
}

.footer-link {
    font-family: 'Roboto', sans-serif;
    font-feature-settings: 'liga' off;
    color: #707A8B;
}

.open-icon {
    color: #707A8B;
    height: 18px;
    width: 18px;
}

.card-container:hover {
    filter: none !important;
    box-shadow: 0 5px 5px 0 rgba(0, 0, 0, 0.05), 0 4px 4px 0 rgba(0, 0, 0, 0.05);
}

* {
    cursor: pointer;
}

.card-divider {
    border-color: var(--fourth-gray-text) !important;
}

.card-divider {
    border-color: var(--box-divider) !important;
}

.apy-box-total-label {
    font-family: 'Roboto', sans-serif;
    text-transform: uppercase;
    font-feature-settings: 'pnum' on, 'lnum' on;
    color: var(--main-gray-text);
}

.apy-box-subtitle-label {
    font-family: 'Roboto', sans-serif;
    text-transform: uppercase;
    font-feature-settings: 'pnum' on, 'lnum' on;
    color: var(--main-gray-text);
}

.insurance-card-info-row {
    border-top: 1px solid var(--main-border) !important;
}

.card-info-label, .card-info-value {
    font-family: 'Roboto', sans-serif;
    color: var(--main-gray-text);
}

.card-about-label {
  font-family: 'Roboto';
  font-style: normal;
  font-weight: 400;
  font-size: 14px;
  line-height: 18px;
  letter-spacing: 0.03em;
  text-transform: uppercase;
}

.tooltip-info {
  position: absolute;
  right: -4px;
  top: 10px;
}

.value-info {
  padding-right: 15px;
}

.info-container {
  position: relative;
}
</style>
