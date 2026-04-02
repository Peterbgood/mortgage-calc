<template>
  <div class="min-h-screen bg-slate-50 py-12 px-4 sm:px-6 lg:px-8 font-sans text-slate-900">
    <div class="max-w-7xl mx-auto">
      <header class="text-center mb-12 relative">
        <h1 class="text-4xl font-extrabold tracking-tight italic text-slate-800">Relocation Cost Comparison</h1>
        <p class="mt-4 text-lg text-slate-600 font-medium uppercase tracking-widest underline decoration-blue-500 underline-offset-8">Annual Cost Variance Analysis</p>
        
        <button @click="resetData" class="mt-8 bg-slate-200 hover:bg-rose-500 hover:text-white text-slate-600 px-6 py-2 rounded-full text-[10px] font-black uppercase tracking-widest transition-all shadow-sm border border-slate-300">
          Reset All to Defaults
        </button>
      </header>

      <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
        <div v-for="(loc, key) in locations" :key="key" 
             class="bg-white rounded-3xl shadow-xl overflow-hidden border border-slate-200 flex flex-col transition-all hover:shadow-2xl">
          
          <div class="bg-slate-900 p-6 text-white text-center">
            <h2 class="text-2xl font-bold uppercase tracking-wider">{{ loc.name }}</h2>
            <p class="text-slate-400 text-[10px] mt-1 font-black tracking-widest uppercase">Monthly Net Surplus</p>
            <p class="text-3xl font-black text-emerald-400 mt-1">{{ formatCurr(getFinalRemainder(loc)) }}</p>
          </div>

          <div class="p-6 space-y-6 flex-grow bg-white">
            <div class="space-y-4">
              <div class="flex justify-between items-end border-b border-slate-100 pb-2">
                <span class="text-[10px] font-black text-slate-400 uppercase">Monthly Gross Pay</span>
                <span class="text-xl font-black text-slate-800">{{ formatCurr(getMonthlyGross(loc)) }}</span>
              </div>
              <div class="grid grid-cols-2 gap-4">
                <div>
                  <label class="block text-[10px] font-black text-slate-400 uppercase mb-1">Wife Salary</label>
                  <input v-model.number="loc.wifeIncome" type="number" class="w-full bg-slate-50 border-slate-200 rounded-lg p-2 text-xs font-bold outline-none focus:ring-2 focus:ring-blue-500">
                </div>
                <div>
                  <label class="block text-[10px] font-black text-slate-400 uppercase mb-1">Husband Salary</label>
                  <input v-model.number="loc.husbandIncome" type="number" class="w-full bg-slate-50 border-slate-200 rounded-lg p-2 text-xs font-bold outline-none focus:ring-2 focus:ring-blue-500">
                </div>
              </div>
            </div>

            <div class="p-4 bg-rose-50 rounded-2xl border border-rose-100 space-y-2">
              <h3 class="text-[10px] font-black text-rose-600 uppercase tracking-widest border-b border-rose-200 pb-1">Paycheck Deductions</h3>
              <div class="flex justify-between text-[11px]">
                <span class="text-slate-500 italic">Fed/FICA (Est)</span>
                <span class="font-bold text-rose-600">-{{ formatCurr(getMonthlyFedTax(loc) + getMonthlyFICA(loc)) }}</span>
              </div>
              <div class="flex justify-between text-[11px]">
                <span class="text-slate-500 italic">Medical (Family)</span>
                <span class="font-bold text-rose-600">-$600</span>
              </div>
              <div v-if="loc.stateTaxRate > 0" class="flex justify-between text-[11px] font-bold text-blue-700">
                <span>{{ loc.name.split(',')[1].trim() }} State Tax</span>
                <span>-{{ formatCurr(getYearlyStateTax(loc) / 12) }}</span>
              </div>
              <div class="pt-2 border-t border-rose-200 flex justify-between items-center font-black text-slate-900">
                <span class="text-[10px] uppercase">Take-Home</span>
                <span class="text-sm">{{ formatCurr(getMonthlyNetAfterTax(loc)) }}</span>
              </div>
            </div>

            <div class="p-4 bg-slate-50 rounded-2xl border border-slate-200">
              <h3 class="text-[10px] font-black text-slate-500 uppercase tracking-widest border-b border-slate-200 pb-2 mb-3">Monthly Bills & Cars</h3>
              <div class="grid grid-cols-2 gap-x-4 gap-y-2">
                <div v-for="(val, bill) in loc.bills" :key="bill">
                  <label class="block text-[9px] font-bold text-slate-400 uppercase">{{ bill }}</label>
                  <input v-model.number="loc.bills[bill]" type="number" class="w-full bg-white border-slate-200 rounded px-1.5 py-1 text-[11px] font-bold outline-none focus:border-blue-400">
                </div>
              </div>
            </div>

            <div class="p-4 bg-blue-50 rounded-2xl border border-blue-100 space-y-3">
              <h3 class="text-[10px] font-black text-blue-600 uppercase tracking-widest border-b border-blue-200 pb-1">Mortgage & Property</h3>
              <div class="grid grid-cols-2 gap-2">
                <div>
                  <label class="block text-[9px] font-bold text-slate-400 uppercase">Home Price</label>
                  <input v-model.number="loc.homePrice" type="number" class="w-full bg-white border-slate-200 rounded p-1 text-[11px] font-bold outline-none">
                </div>
                <div>
                  <label class="block text-[9px] font-bold text-slate-400 uppercase">Down Pmt</label>
                  <input v-model.number="loc.downPayment" type="number" class="w-full bg-white border-slate-200 rounded p-1 text-[11px] font-bold outline-none">
                </div>
                <div>
                  <label class="block text-[9px] font-bold text-slate-400 uppercase">Rate %</label>
                  <input v-model.number="loc.rate" type="number" step="0.1" class="w-full bg-white border-slate-200 rounded p-1 text-[11px] font-bold outline-none">
                </div>
                <div class="flex flex-col justify-end">
                   <div class="flex justify-between items-center text-[10px]">
                      <span class="font-bold text-slate-400 uppercase">P&I:</span>
                      <span class="font-bold text-slate-900">{{ formatCurr(calcPI(loc)) }}</span>
                   </div>
                </div>
                <div>
                  <label class="block text-[9px] font-bold text-slate-400 uppercase">Yearly Tax</label>
                  <input v-model.number="loc.fixedPropertyTax" type="number" class="w-full bg-white border-slate-200 rounded p-1 text-[11px] font-bold outline-none">
                </div>
                <div>
                  <label class="block text-[9px] font-bold text-slate-400 uppercase">Yearly Ins.</label>
                  <input v-model.number="loc.fixedHomeInsurance" type="number" class="w-full bg-white border-slate-200 rounded p-1 text-[11px] font-bold outline-none">
                </div>
              </div>
              <div class="pt-2 border-t border-blue-200 flex justify-between items-center font-black">
                <span class="text-blue-800 uppercase text-[10px]">Total House Payment</span>
                <span class="text-sm text-blue-900">{{ formatCurr(getFullMortgagePayment(loc)) }}</span>
              </div>
            </div>
          </div>

          <div class="bg-slate-100 p-6 border-t border-slate-200 mt-auto space-y-4">
            <div v-if="key !== 'knoxville'">
              <div class="flex justify-between items-center">
                <p class="text-[10px] uppercase font-black text-slate-500">Monthly Difference</p>
                <span :class="getMonthlyDiff(loc) >= 0 ? 'text-emerald-600' : 'text-rose-600'" class="text-xl font-black">
                  {{ getMonthlyDiff(loc) >= 0 ? '+' : '' }}{{ formatCurr(getMonthlyDiff(loc)) }}
                </span>
              </div>
              <div class="pt-4 border-t border-slate-300">
                <div class="flex justify-between items-center">
                  <p class="text-[11px] uppercase font-black text-slate-700">Annual Cost Impact</p>
                  <span :class="getMonthlyDiff(loc) >= 0 ? 'text-emerald-600' : 'text-rose-600'" class="text-2xl font-black italic">
                    {{ getMonthlyDiff(loc) >= 0 ? 'Save ' : 'Cost ' }}{{ formatCurr(Math.abs(getMonthlyDiff(loc) * 12)) }}
                  </span>
                </div>
                <p class="text-[9px] text-slate-400 font-bold uppercase mt-1">Difference vs Knox Baseline over 12 months</p>
              </div>
            </div>
            <div v-else class="text-center py-8">
              <span class="text-[10px] uppercase font-black text-slate-400 tracking-[0.2em] bg-white border border-slate-200 px-6 py-2 rounded-full">Knoxville Baseline</span>
            </div>
          </div>

        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { reactive, computed } from 'vue';

const baseBills = {
  groceries: 800, electric: 350, att: 150, water: 150, 
  internet: 65, philo: 35, lifeIns: 90, amazon: 20, 
  netflix: 20, bugPeople: 35, volvoGas: 100, chickfila: 160, 
  pets: 70, beer: 208, model3: 490, modelY: 415
};

interface LocationData {
  name: string; wifeIncome: number; husbandIncome: number;
  homePrice: number; downPayment: number; rate: number;
  fixedPropertyTax: number; fixedHomeInsurance: number;
  stateTaxRate: number; bills: Record<string, number>;
}

const defaultData: Record<string, LocationData> = {
  knoxville: {
    name: 'Knoxville, TN', wifeIncome: 100000, husbandIncome: 100000,
    homePrice: 450000, downPayment: 210000, rate: 5.0,
    fixedPropertyTax: 2940, fixedHomeInsurance: 800,
    stateTaxRate: 0, 
    bills: { ...baseBills, carInsurance: 300 }
  },
  charlotte: {
    name: 'Charlotte, NC', wifeIncome: 100000, husbandIncome: 100000,
    homePrice: 450000, downPayment: 150000, rate: 6.3,
    fixedPropertyTax: 4200, fixedHomeInsurance: 1300,
    stateTaxRate: 0.045, 
    bills: { ...baseBills, carInsurance: 360 }
  },
  tampa: {
    name: 'Tampa, FL', wifeIncome: 100000, husbandIncome: 100000,
    homePrice: 450000, downPayment: 150000, rate: 6.3,
    fixedPropertyTax: 9000, fixedHomeInsurance: 4000, 
    stateTaxRate: 0, 
    bills: { ...baseBills, carInsurance: 550 }
  }
};

const locations = reactive<Record<string, LocationData>>(JSON.parse(JSON.stringify(defaultData)));
const resetData = () => { Object.keys(defaultData).forEach(k => Object.assign(locations[k], JSON.parse(JSON.stringify(defaultData[k])))); };

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

const getMonthlyBillsTotal = (l: LocationData) => Object.values(l.bills).reduce((a, b) => a + b, 0);

const getMonthlyNetAfterTax = (l: LocationData): number => {
  return getMonthlyGross(l) - getMonthlyFedTax(l) - getMonthlyFICA(l) - (getYearlyStateTax(l)/12) - 600;
};

const getFullMortgagePayment = (l: LocationData) => calcPI(l) + (l.fixedPropertyTax / 12) + (l.fixedHomeInsurance / 12);
const getFinalRemainder = (l: LocationData) => getMonthlyNetAfterTax(l) - getFullMortgagePayment(l) - getMonthlyBillsTotal(l);

const knoxFinal = computed(() => getFinalRemainder(locations['knoxville']!));
const getMonthlyDiff = (l: LocationData) => getFinalRemainder(l) - knoxFinal.value;
const formatCurr = (v: number) => new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD', maximumFractionDigits: 0 }).format(v);
</script>