<template>
  <div class="min-h-screen bg-slate-50 py-12 px-4 sm:px-6 lg:px-8 font-sans text-slate-900">
    <div class="max-w-7xl mx-auto">
      <header class="text-center mb-12">
        <h1 class="text-4xl font-extrabold tracking-tight">Relocation Cost Comparison</h1>
        <p class="mt-4 text-lg text-slate-600 font-medium uppercase tracking-widest">Knoxville vs. Charlotte vs. Tampa</p>
      </header>

      <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
        <div v-for="(loc, key) in locations" :key="key" 
             class="bg-white rounded-3xl shadow-xl overflow-hidden border border-slate-200 flex flex-col transition-all hover:shadow-2xl">
          
          <div class="bg-slate-900 p-6 text-white text-center">
            <h2 class="text-2xl font-bold uppercase tracking-wider">{{ loc.name }}</h2>
            <p class="text-slate-400 text-[10px] mt-1 font-black tracking-widest">ESTIMATED MONTHLY NET</p>
            <p class="text-3xl font-black text-emerald-400 mt-1">{{ formatCurr(getNetRemaining(loc)) }}</p>
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
                <label class="block text-[10px] font-black text-slate-400 uppercase mb-1">Down Payment</label>
                <input v-model.number="loc.downPayment" type="number" class="w-full bg-slate-50 border-slate-200 rounded-lg p-2 text-sm font-bold focus:ring-2 focus:ring-blue-500 outline-none">
              </div>
            </div>

            <div>
              <label class="block text-[10px] font-black text-slate-400 uppercase mb-1">Interest Rate %</label>
              <input v-model.number="loc.rate" type="number" step="0.1" class="w-full bg-slate-50 border-slate-200 rounded-lg p-2 text-sm font-bold focus:ring-2 focus:ring-blue-500 outline-none">
            </div>

            <div class="mt-6 p-5 bg-blue-50 rounded-2xl border border-blue-100 space-y-3">
              <h3 class="text-[11px] font-black text-blue-600 uppercase tracking-widest border-b border-blue-200 pb-2">Monthly Mortgage (PITI)</h3>
              
              <div class="flex justify-between text-sm font-medium">
                <span class="text-slate-600">Principal & Interest</span>
                <span>{{ formatCurr(calcPI(loc)) }}</span>
              </div>
              
              <div class="flex justify-between text-sm font-medium">
                <span class="text-slate-600">Monthly Taxes</span>
                <span>{{ formatCurr(getYearlyPropertyTax(loc) / 12) }}</span>
              </div>

              <div class="flex justify-between text-sm font-medium">
                <span class="text-slate-600">Monthly Home Ins.</span>
                <span>{{ formatCurr(loc.homeInsurance / 12) }}</span>
              </div>

              <div class="pt-2 border-t border-blue-200 flex justify-between items-center">
                <span class="text-xs font-black text-slate-700 uppercase leading-none">Full Payment</span>
                <span class="text-xl font-black text-blue-700">{{ formatCurr(getFullMortgagePayment(loc)) }}</span>
              </div>
            </div>

            <div class="mt-4 p-4 bg-slate-900 rounded-2xl space-y-2 shadow-inner">
              <h3 class="text-[10px] font-black text-slate-400 uppercase tracking-widest border-b border-slate-700 pb-2 mb-2">Other Annual Costs</h3>
              
              <div class="flex justify-between text-xs">
                <span class="text-slate-500">State Income Tax</span>
                <span class="font-bold text-rose-400">{{ formatCurr(getYearlyStateTax(loc)) }}</span>
              </div>

              <div class="flex justify-between text-xs">
                <span class="text-slate-500">3-Car Auto Policy</span>
                <span class="font-bold text-slate-200">{{ formatCurr(loc.carInsurance * 12) }}</span>
              </div>
            </div>
          </div>

          <div class="bg-slate-100 p-6 border-t border-slate-200 mt-auto">
            <div v-if="key !== 'knoxville'" class="space-y-4">
              <div class="flex justify-between items-center">
                <div>
                  <p class="text-[10px] uppercase font-black text-slate-400 leading-none">Cost Difference</p>
                  <p class="text-xs text-slate-500 font-bold mt-1">Expenses / Month</p>
                </div>
                <span :class="getExpenseDiff(loc) <= 0 ? 'text-emerald-600' : 'text-rose-600'" class="text-2xl font-black italic">
                  {{ getExpenseDiff(loc) > 0 ? '+' : '' }}{{ formatCurr(getExpenseDiff(loc)) }}
                </span>
              </div>
              <div class="pt-3 border-t border-slate-200 flex justify-between items-center">
                <p class="text-[10px] uppercase font-black text-slate-400">Annual Expense Impact</p>
                <span :class="getExpenseDiff(loc) <= 0 ? 'text-emerald-700' : 'text-rose-700'" class="text-sm font-black text-right">
                   {{ getExpenseDiff(loc) > 0 ? '+' : '' }}{{ formatCurr(getExpenseDiff(loc) * 12) }} / yr
                </span>
              </div>
              <p class="text-[9px] text-slate-400 italic text-center leading-tight mt-2">
                Impact tracks the difference in taxes and bills compared to Knoxville, ignoring total income changes.
              </p>
            </div>
            <div v-else class="text-center py-8">
              <span class="text-[10px] uppercase font-black text-slate-400 tracking-widest bg-slate-200 px-3 py-1 rounded-full">Baseline Location</span>
            </div>
          </div>

        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { reactive } from 'vue';

interface LocationData {
  name: string;
  wifeIncome: number;
  husbandIncome: number;
  homePrice: number;
  downPayment: number;
  rate: number;
  taxRate: number;
  stateTax: number;
  homeInsurance: number;
  carInsurance: number; 
}

const locations = reactive<Record<string, LocationData>>({
  knoxville: {
    name: 'Knoxville, TN',
    wifeIncome: 100000,
    husbandIncome: 100000,
    homePrice: 450000,
    downPayment: 210000, 
    rate: 6.3,
    taxRate: 0.0039, 
    stateTax: 0,
    homeInsurance: 850,
    carInsurance: 300
  },
  charlotte: {
    name: 'Charlotte, NC',
    wifeIncome: 100000,
    husbandIncome: 100000,
    homePrice: 450000,
    downPayment: 150000, 
    rate: 6.3,
    taxRate: 0.0095, 
    stateTax: 0.045, 
    homeInsurance: 1300,
    carInsurance: 380
  },
  tampa: {
    name: 'Tampa, FL',
    wifeIncome: 100000,
    husbandIncome: 100000,
    homePrice: 450000,
    downPayment: 150000, 
    rate: 6.3,
    taxRate: 0.012, 
    stateTax: 0,
    homeInsurance: 4800, 
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

const getYearlyPropertyTax = (loc: LocationData) => loc.homePrice * loc.taxRate;
const getYearlyStateTax = (loc: LocationData) => (loc.wifeIncome + loc.husbandIncome) * loc.stateTax;

const getFullMortgagePayment = (loc: LocationData) => {
  const monthlyTax = getYearlyPropertyTax(loc) / 12;
  const monthlyIns = loc.homeInsurance / 12;
  return calcPI(loc) + monthlyTax + monthlyIns;
};

// Calculates total monthly drain (Expenses + State Tax)
const getTotalMonthlyExpenses = (loc: LocationData): number => {
  const monthlyStateTax = getYearlyStateTax(loc) / 12;
  return getFullMortgagePayment(loc) + loc.carInsurance + monthlyStateTax;
};

// Only compares the "drain" (expenses) relative to Knoxville
const getExpenseDiff = (loc: LocationData): number => {
  return getTotalMonthlyExpenses(loc) - getTotalMonthlyExpenses(locations.knoxville);
};

const getNetMonthlyIncome = (loc: LocationData): number => {
  const combined = loc.wifeIncome + loc.husbandIncome;
  const federalTax = 0.15; 
  const stateTaxAmt = combined * loc.stateTax;
  const netIncome = (combined - (combined * federalTax) - stateTaxAmt) / 12;
  return netIncome - getFullMortgagePayment(loc) - loc.carInsurance;
};

// For the emerald number in the top header
const getNetRemaining = (loc: LocationData) => {
  const combined = loc.wifeIncome + loc.husbandIncome;
  const federalTax = 0.15;
  const stateTaxAmt = combined * loc.stateTax;
  const monthlyNetIn = (combined - (combined * federalTax) - stateTaxAmt) / 12;
  return monthlyNetIn - getFullMortgagePayment(loc) - loc.carInsurance;
};

const formatCurr = (val: number) => {
  return new Intl.NumberFormat('en-US', { 
    style: 'currency', 
    currency: 'USD',
    maximumFractionDigits: 0 
  }).format(val);
};
</script>