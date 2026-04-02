<template>
  <div class="min-h-screen bg-slate-50 py-12 px-4 sm:px-6 lg:px-8 font-sans text-slate-900">
    <div class="max-w-7xl mx-auto">
      <header class="text-center mb-12 relative">
        <h1 class="text-4xl font-extrabold tracking-tight italic text-slate-800">Heisenberg's Empire Expansion</h1>
        <p class="mt-4 text-lg text-slate-600 font-medium uppercase tracking-widest underline decoration-blue-500 underline-offset-8">Regional Overhead & Money Laundering Analysis</p>
        
        <button @click="resetData" class="mt-8 bg-slate-200 hover:bg-rose-500 hover:text-white text-slate-600 px-6 py-2 rounded-full text-[10px] font-black uppercase tracking-widest transition-all shadow-sm border border-slate-300">
          Reset to Blue Sky Defaults
        </button>
      </header>

      <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
        <div v-for="(loc, key) in locations" :key="key" 
             class="bg-white rounded-3xl shadow-xl overflow-hidden border border-slate-200 flex flex-col transition-all hover:shadow-2xl">
          
          <div class="bg-slate-900 p-6 text-white text-center">
            <h2 class="text-2xl font-bold uppercase tracking-wider">{{ loc.name }}</h2>
            <p class="text-slate-400 text-[10px] mt-1 font-black tracking-widest uppercase">Monthly Net Surplus (Pre-Barrel)</p>
            <p class="text-3xl font-black text-emerald-400 mt-1">{{ formatCurr(getFinalRemainder(loc)) }}</p>
          </div>

          <div class="p-6 space-y-6 flex-grow bg-white">
            <div class="space-y-4">
              <div class="flex justify-between items-end border-b border-slate-100 pb-2">
                <span class="text-[10px] font-black text-slate-400 uppercase">Monthly Distribution Gross</span>
                <span class="text-xl font-black text-slate-800">{{ formatCurr(getMonthlyGross(loc)) }}</span>
              </div>
              <div class="grid grid-cols-2 gap-4">
                <div>
                  <label class="block text-[10px] font-black text-slate-400 uppercase mb-1">Skyler's Cut</label>
                  <input v-model.number="loc.wifeIncome" type="number" class="w-full bg-slate-50 border-slate-200 rounded-lg p-2 text-xs font-bold outline-none focus:ring-2 focus:ring-blue-500">
                </div>
                <div>
                  <label class="block text-[10px] font-black text-slate-400 uppercase mb-1">Walt's Cut</label>
                  <input v-model.number="loc.husbandIncome" type="number" class="w-full bg-slate-50 border-slate-200 rounded-lg p-2 text-xs font-bold outline-none focus:ring-2 focus:ring-blue-500">
                </div>
              </div>
            </div>

            <div class="p-4 bg-rose-50 rounded-2xl border border-rose-100 space-y-2">
              <h3 class="text-[10px] font-black text-rose-600 uppercase tracking-widest border-b border-rose-200 pb-1">DEA / IRS Deductions</h3>
              <div class="flex justify-between text-[11px]">
                <span class="text-slate-500 italic">Federal "Taxes" (Est)</span>
                <span class="font-bold text-rose-600">-{{ formatCurr(getMonthlyFedTax(loc) + getMonthlyFICA(loc)) }}</span>
              </div>
              <div class="flex justify-between text-[11px]">
                <span class="text-slate-500 italic">Saul's Legal Retainer</span>
                <span class="font-bold text-rose-600">-$600</span>
              </div>
              <div v-if="loc.stateTaxRate > 0" class="flex justify-between text-[11px] font-bold text-blue-700">
                <span>{{ loc.name.split(',')[1]?.trim() || 'State' }} Protection</span>
                <span>-{{ formatCurr(getYearlyStateTax(loc) / 12) }}</span>
              </div>
              <div class="pt-2 border-t border-rose-200 flex justify-between items-center font-black text-slate-900">
                <span class="text-[10px] uppercase">Clean Take-Home</span>
                <span class="text-sm">{{ formatCurr(getMonthlyNetAfterTax(loc)) }}</span>
              </div>
            </div>

            <div class="p-4 bg-slate-50 rounded-2xl border border-slate-200">
              <h3 class="text-[10px] font-black text-slate-500 uppercase tracking-widest border-b border-slate-200 pb-2 mb-3">Operating Expenses</h3>
              <div class="grid grid-cols-2 gap-x-4 gap-y-2">
                <div v-for="(val, bill) in loc.bills" :key="bill">
                  <label class="block text-[9px] font-bold text-slate-400 uppercase">{{ bill }}</label>
                  <input v-model.number="loc.bills[bill]" type="number" class="w-full bg-white border-slate-200 rounded px-1.5 py-1 text-[11px] font-bold outline-none focus:border-blue-400">
                </div>
              </div>
            </div>

            <div class="p-4 bg-blue-50 rounded-2xl border border-blue-100 space-y-3">
              <h3 class="text-[10px] font-black text-blue-600 uppercase tracking-widest border-b border-blue-200 pb-1">Safehouse & Property</h3>
              <div class="grid grid-cols-2 gap-2">
                <div>
                  <label class="block text-[9px] font-bold text-slate-400 uppercase">Resale Value</label>
                  <input v-model.number="loc.homePrice" type="number" class="w-full bg-white border-slate-200 rounded p-1 text-[11px] font-bold outline-none">
                </div>
                <div>
                  <label class="block text-[9px] font-bold text-slate-400 uppercase">Cash Down</label>
                  <input v-model.number="loc.downPayment" type="number" class="w-full bg-white border-slate-200 rounded p-1 text-[11px] font-bold outline-none">
                </div>
                <div>
                  <label class="block text-[9px] font-bold text-slate-400 uppercase">APR %</label>
                  <input v-model.number="loc.rate" type="number" step="0.1" class="w-full bg-white border-slate-200 rounded p-1 text-[11px] font-bold outline-none">
                </div>
                <div class="flex flex-col justify-end">
                   <div class="flex justify-between items-center text-[10px]">
                      <span class="font-bold text-slate-400 uppercase">P&I:</span>
                      <span class="font-bold text-slate-900">{{ formatCurr(calcPI(loc)) }}</span>
                   </div>
                </div>
                <div>
                  <label class="block text-[9px] font-bold text-slate-400 uppercase">Prop. Tax</label>
                  <input v-model.number="loc.fixedPropertyTax" type="number" class="w-full bg-white border-slate-200 rounded p-1 text-[11px] font-bold outline-none">
                </div>
                <div>
                  <label class="block text-[9px] font-bold text-slate-400 uppercase">Home Ins.</label>
                  <input v-model.number="loc.fixedHomeInsurance" type="number" class="w-full bg-white border-slate-200 rounded p-1 text-[11px] font-bold outline-none">
                </div>
              </div>
              <div class="pt-2 border-t border-blue-200 flex justify-between items-center font-black">
                <span class="text-blue-800 uppercase text-[10px]">Total Safehouse Pmt</span>
                <span class="text-sm text-blue-900">{{ formatCurr(getFullMortgagePayment(loc)) }}</span>
              </div>
            </div>
          </div>

          <div class="bg-slate-100 p-6 border-t border-slate-200 mt-auto space-y-4">
            <div v-if="key !== 'abq'">
              <div class="flex justify-between items-center">
                <p class="text-[10px] uppercase font-black text-slate-500">Monthly Variance</p>
                <span :class="getMonthlyDiff(loc) >= 0 ? 'text-emerald-600' : 'text-rose-600'" class="text-xl font-black">
                  {{ getMonthlyDiff(loc) >= 0 ? '+' : '' }}{{ formatCurr(getMonthlyDiff(loc)) }}
                </span>
              </div>
              <div class="pt-4 border-t border-slate-300">
                <div class="flex justify-between items-center">
                  <p class="text-[11px] uppercase font-black text-slate-700">Annual Empire Growth</p>
                  <span :class="getMonthlyDiff(loc) >= 0 ? 'text-emerald-600' : 'text-rose-600'" class="text-2xl font-black italic">
                    {{ getMonthlyDiff(loc) >= 0 ? 'Save ' : 'Loss ' }}{{ formatCurr(Math.abs(getMonthlyDiff(loc) * 12)) }}
                  </span>
                </div>
              </div>
            </div>
            <div v-else class="text-center py-8">
              <span class="text-[10px] uppercase font-black text-slate-400 tracking-[0.2em] bg-white border border-slate-200 px-6 py-2 rounded-full">ABQ Baseline</span>
            </div>
          </div>

        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { reactive, computed } from 'vue';

interface Bills {
  carWashSupplies: number; pestControl: number; breakfast: number; attorney: number;
  security: number; storageUnits: number; pollosHermanos: number; gas: number;
  [key: string]: number;
}

interface LocationData {
  name: string; wifeIncome: number; husbandIncome: number;
  homePrice: number; downPayment: number; rate: number;
  fixedPropertyTax: number; fixedHomeInsurance: number;
  stateTaxRate: number; bills: Bills;
}

const baseBills: Bills = {
  carWashSupplies: 800, pestControl: 350, breakfast: 150, attorney: 500, 
  security: 600, storageUnits: 250, pollosHermanos: 160, gas: 100
};

const defaultData: Record<string, LocationData> = {
  abq: {
    name: 'Albuquerque, NM', wifeIncome: 120000, husbandIncome: 150000,
    homePrice: 500000, downPayment: 200000, rate: 5.5,
    fixedPropertyTax: 4500, fixedHomeInsurance: 1200,
    stateTaxRate: 0.05, bills: { ...baseBills }
  },
  newhampshire: {
    name: 'Snowy Cabin, NH', wifeIncome: 0, husbandIncome: 80000,
    homePrice: 350000, downPayment: 350000, rate: 0,
    fixedPropertyTax: 7000, fixedHomeInsurance: 2000,
    stateTaxRate: 0, bills: { ...baseBills, carWashSupplies: 0, pollosHermanos: 20 } 
  },
  elpaso: {
    name: 'El Paso, TX', wifeIncome: 100000, husbandIncome: 100000,
    homePrice: 450000, downPayment: 150000, rate: 6.8,
    fixedPropertyTax: 9500, fixedHomeInsurance: 3000, 
    stateTaxRate: 0, bills: { ...baseBills, security: 1200 }
  }
};

const locations = reactive<Record<string, LocationData>>(JSON.parse(JSON.stringify(defaultData)));

const resetData = () => { 
  Object.keys(defaultData).forEach(k => {
    Object.assign(locations[k]!, JSON.parse(JSON.stringify(defaultData[k]!)));
  }); 
};

const getMonthlyGross = (l: LocationData) => (l.wifeIncome + l.husbandIncome) / 12;
const getMonthlyFedTax = (l: LocationData) => getMonthlyGross(l) * 0.15;
const getMonthlyFICA = (l: LocationData) => getMonthlyGross(l) * 0.0765;
const getYearlyStateTax = (l: LocationData) => (l.wifeIncome + l.husbandIncome) * l.stateTaxRate;

const calcPI = (l: LocationData): number => {
  const p = l.homePrice - l.downPayment;
  if (p <= 0) return 0;
  const r = (l.rate / 100) / 12;
  const n = 360;
  return (p * r * Math.pow(1 + r, n)) / (Math.pow(1 + r, n) - 1);
};

const getMonthlyBillsTotal = (l: LocationData) => {
  return Object.values(l.bills).reduce((a, b) => a + (Number(b) || 0), 0);
};

const getMonthlyNetAfterTax = (l: LocationData): number => {
  return getMonthlyGross(l) - getMonthlyFedTax(l) - getMonthlyFICA(l) - (getYearlyStateTax(l)/12) - 600;
};

const getFullMortgagePayment = (l: LocationData) => calcPI(l) + (l.fixedPropertyTax / 12) + (l.fixedHomeInsurance / 12);
const getFinalRemainder = (l: LocationData) => getMonthlyNetAfterTax(l) - getFullMortgagePayment(l) - getMonthlyBillsTotal(l);

const abqFinal = computed(() => getFinalRemainder(locations['abq']!));
const getMonthlyDiff = (l: LocationData) => getFinalRemainder(l) - abqFinal.value;
const formatCurr = (v: number) => new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD', maximumFractionDigits: 0 }).format(v);
</script>