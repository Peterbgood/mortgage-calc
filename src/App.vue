<template>
  <div class="min-h-screen bg-slate-50 py-12 px-4 sm:px-6 lg:px-8 font-sans text-slate-900">
    <div class="max-w-7xl mx-auto">
      <header class="text-center mb-12">
        <h1 class="text-4xl font-extrabold tracking-tight italic text-slate-800">Relocation Cost Comparison</h1>
        <p class="mt-4 text-lg text-slate-600 font-medium uppercase tracking-widest underline decoration-blue-500 underline-offset-8">Financial Breakdown</p>
      </header>

      <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
        <div v-for="(loc, key) in locations" :key="key" 
             class="bg-white rounded-3xl shadow-xl overflow-hidden border border-slate-200 flex flex-col transition-all hover:shadow-2xl">
          
          <div class="bg-slate-900 p-6 text-white text-center">
            <h2 class="text-2xl font-bold uppercase tracking-wider">{{ loc.name }}</h2>
            <p class="text-slate-400 text-[10px] mt-1 font-black tracking-widest uppercase">Monthly Cash Remaining</p>
            <p class="text-3xl font-black text-emerald-400 mt-1">{{ formatCurr(getFinalRemainder(loc)) }}</p>
          </div>

          <div class="p-6 space-y-4 flex-grow bg-white">
            <div class="grid grid-cols-2 gap-4">
              <div>
                <label class="block text-[10px] font-black text-slate-400 uppercase mb-1">Wife Income</label>
                <input v-model.number="loc.wifeIncome" type="number" class="w-full bg-slate-50 border-slate-200 rounded-lg p-2 text-sm font-bold focus:ring-2 focus:ring-blue-500 outline-none">
              </div>
              <div>
                <label class="block text-[10px] font-black text-slate-400 uppercase mb-1">Husband Income</label>
                <input v-model.number="loc.husbandIncome" type="number" class="w-full bg-slate-50 border-slate-200 rounded-lg p-2 text-sm font-bold focus:ring-2 focus:ring-blue-500 outline-none">
              </div>
            </div>

            <div class="grid grid-cols-2 gap-4">
              <div>
                <label class="block text-[10px] font-black text-slate-400 uppercase mb-1">Home Price</label>
                <input v-model.number="loc.homePrice" type="number" class="w-full bg-slate-50 border-slate-200 rounded-lg p-2 text-sm font-bold focus:ring-2 focus:ring-blue-500 outline-none">
              </div>
              <div>
                <label class="block text-[10px] font-black text-slate-400 uppercase mb-1">Down Pmt</label>
                <input v-model.number="loc.downPayment" type="number" class="w-full bg-slate-50 border-slate-200 rounded-lg p-2 text-sm font-bold focus:ring-2 focus:ring-blue-500 outline-none">
              </div>
            </div>

            <div>
              <label class="block text-[10px] font-black text-slate-400 uppercase mb-1">Interest Rate %</label>
              <input v-model.number="loc.rate" type="number" step="0.1" class="w-full bg-slate-50 border-slate-200 rounded-lg p-2 text-sm font-bold focus:ring-2 focus:ring-blue-500 outline-none">
            </div>

            <div class="p-4 bg-rose-50 rounded-2xl border border-rose-100 space-y-2">
              <div class="flex justify-between items-center border-b border-rose-200 pb-2 mb-1">
                <h3 class="text-[10px] font-black text-rose-600 uppercase tracking-widest">Tax Deductions</h3>
                <span class="text-[10px] font-bold text-rose-800 bg-rose-100 px-2 py-0.5 rounded">Gross: {{ formatCurr((loc.wifeIncome + loc.husbandIncome) / 12) }}</span>
              </div>
              <div class="flex justify-between text-xs">
                <span class="text-slate-600">Federal Tax (15%)</span>
                <span class="font-bold text-rose-600">-{{ formatCurr((loc.wifeIncome + loc.husbandIncome) * 0.15 / 12) }}</span>
              </div>
              <div v-if="loc.stateTaxRate > 0" class="flex justify-between text-xs">
                <span class="text-slate-600">State Tax ({{(loc.stateTaxRate * 100).toFixed(1)}}%)</span>
                <span class="font-bold text-rose-600">-{{ formatCurr(getYearlyStateTax(loc) / 12) }}</span>
              </div>
              <div class="flex justify-between text-xs">
                <span class="text-slate-600">Sales Tax Est.</span>
                <span class="font-bold text-rose-600">-{{ formatCurr(getMonthlySalesTax(loc)) }}</span>
              </div>
              <div class="pt-2 border-t border-rose-200 flex justify-between items-center">
                <span class="text-xs font-black text-slate-700 uppercase">Monthly Net Take-home</span>
                <span class="text-sm font-black text-slate-900">{{ formatCurr(getMonthlyNetAfterTax(loc)) }}</span>
              </div>
            </div>

            <div class="p-4 bg-blue-50 rounded-2xl border border-blue-100 space-y-2 shadow-sm">
              <h3 class="text-[10px] font-black text-blue-600 uppercase tracking-widest border-b border-blue-200 pb-2">Monthly Fixed Costs</h3>
              
              <div class="space-y-1.5 pb-2">
                <div class="flex justify-between text-xs">
                  <span class="text-slate-600 italic">P&I Payment</span>
                  <span class="font-bold text-slate-900">{{ formatCurr(calcPI(loc)) }}</span>
                </div>
                <div class="flex justify-between text-xs">
                  <span class="text-slate-600 italic">Prop. Tax (Escrow)</span>
                  <span class="font-bold text-slate-900">{{ formatCurr(loc.fixedPropertyTax / 12) }}</span>
                </div>
                <div class="flex justify-between text-xs">
                  <span class="text-slate-600 italic">Home Ins. (Escrow)</span>
                  <span class="font-bold text-slate-900">{{ formatCurr(loc.fixedHomeInsurance / 12) }}</span>
                </div>
                <div class="flex justify-between items-center pt-1 border-t border-blue-200 border-dashed">
                  <span class="text-[10px] font-black text-blue-700 uppercase">Total House Payment</span>
                  <span class="text-xs font-black text-blue-800">{{ formatCurr(getFullMortgagePayment(loc)) }}</span>
                </div>
              </div>

              <div class="flex justify-between text-xs pt-2 border-t border-blue-200">
                <span class="text-slate-600 italic">3-Car Auto Policy</span>
                <span class="font-bold text-slate-900">{{ formatCurr(loc.carInsurance) }}</span>
              </div>

              <div class="pt-2 border-t-2 border-blue-300 flex justify-between items-center">
                <span class="text-xs font-black text-blue-900 uppercase">Total Monthly Bills</span>
                <span class="text-sm font-black text-blue-900">{{ formatCurr(getTotalMonthlyBills(loc)) }}</span>
              </div>
            </div>
          </div>

          <div class="bg-slate-100 p-6 border-t border-slate-200 mt-auto">
            <div v-if="key !== 'knoxville'" class="space-y-4">
              <div class="flex justify-between items-center">
                <div>
                  <p class="text-[10px] uppercase font-black text-slate-400 leading-none">Vs. Knoxville Baseline</p>
                  <p class="text-xs text-slate-500 font-bold mt-1 italic">Difference / Month</p>
                </div>
                <span :class="getMonthlyDiff(loc) >= 0 ? 'text-emerald-600' : 'text-rose-600'" class="text-2xl font-black italic">
                  {{ getMonthlyDiff(loc) >= 0 ? '+' : '' }}{{ formatCurr(getMonthlyDiff(loc)) }}
                </span>
              </div>

              <div class="pt-3 border-t border-slate-200 flex justify-between items-center">
                <p class="text-[10px] uppercase font-black text-slate-400">Total Yearly Impact</p>
                <span :class="getMonthlyDiff(loc) >= 0 ? 'text-emerald-700' : 'text-rose-700'" class="text-sm font-black text-right">
                  {{ getMonthlyDiff(loc) >= 0 ? '+' : '' }}{{ formatCurr(getMonthlyDiff(loc) * 12) }} / yr
                </span>
              </div>
            </div>
            <div v-else class="text-center py-10">
              <span class="text-[10px] uppercase font-black text-slate-400 tracking-widest bg-slate-200 px-4 py-2 rounded-full border border-slate-300">Baseline Location</span>
            </div>
          </div>

        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { reactive, computed } from 'vue';

interface LocationData {
  name: string;
  wifeIncome: number;
  husbandIncome: number;
  homePrice: number;
  downPayment: number;
  rate: number;
  fixedPropertyTax: number;
  fixedHomeInsurance: number;
  stateTaxRate: number;
  salesTaxRate: number;
  carInsurance: number; 
}

const locations = reactive<Record<string, LocationData>>({
  knoxville: {
    name: 'Knoxville, TN',
    wifeIncome: 100000,
    husbandIncome: 100000,
    homePrice: 450000,
    downPayment: 210000, 
    rate: 5.0,
    fixedPropertyTax: 1200, 
    fixedHomeInsurance: 800,
    stateTaxRate: 0,
    salesTaxRate: 0.0925,
    carInsurance: 300
  },
  charlotte: {
    name: 'Charlotte, NC',
    wifeIncome: 100000,
    husbandIncome: 100000,
    homePrice: 450000,
    downPayment: 150000, 
    rate: 6.3,
    fixedPropertyTax: 4200, 
    fixedHomeInsurance: 1300,
    stateTaxRate: 0.045, 
    salesTaxRate: 0.0725,
    carInsurance: 380
  },
  tampa: {
    name: 'Tampa, FL',
    wifeIncome: 100000,
    husbandIncome: 100000,
    homePrice: 450000,
    downPayment: 150000, 
    rate: 6.3,
    fixedPropertyTax: 9000, 
    fixedHomeInsurance: 4000, 
    stateTaxRate: 0,
    salesTaxRate: 0.075,
    carInsurance: 580
  }
});

const calcPI = (loc: LocationData): number => {
  const principal = loc.homePrice - loc.downPayment;
  if (principal <= 0) return 0;
  const monthlyRate = (loc.rate / 100) / 12;
  const n = 30 * 12;
  return (principal * monthlyRate * Math.pow(1 + monthlyRate, n)) / (Math.pow(1 + monthlyRate, n) - 1);
};

const getYearlyStateTax = (loc: LocationData) => (loc.wifeIncome + loc.husbandIncome) * loc.stateTaxRate;

const getMonthlySalesTax = (loc: LocationData) => {
  const discretionary = (loc.wifeIncome + loc.husbandIncome) * 0.30;
  return (discretionary * loc.salesTaxRate) / 12;
};

const getMonthlyNetAfterTax = (loc: LocationData): number => {
  const gross = (loc.wifeIncome + loc.husbandIncome) / 12;
  const fed = gross * 0.15;
  const state = (getYearlyStateTax(loc) / 12);
  const sales = getMonthlySalesTax(loc);
  return gross - fed - state - sales;
};

const getFullMortgagePayment = (loc: LocationData) => {
  return calcPI(loc) + (loc.fixedPropertyTax / 12) + (loc.fixedHomeInsurance / 12);
};

const getTotalMonthlyBills = (loc: LocationData) => {
  return getFullMortgagePayment(loc) + loc.carInsurance;
};

const getFinalRemainder = (loc: LocationData) => {
  return getMonthlyNetAfterTax(loc) - getTotalMonthlyBills(loc);
};

const knoxFinal = computed(() => getFinalRemainder(locations['knoxville']!));
const getMonthlyDiff = (loc: LocationData) => getFinalRemainder(loc) - knoxFinal.value;

const formatCurr = (val: number) => {
  return new Intl.NumberFormat('en-US', { 
    style: 'currency', currency: 'USD', maximumFractionDigits: 0 
  }).format(val);
};
</script>