# devil-fruit-game
Game sưu tầm trái ác quỷ
import React, { useEffect, useMemo, useRef, useState } from "react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Progress } from "@/components/ui/progress";
import { Badge } from "@/components/ui/badge";
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";
import { motion } from "framer-motion";
import {
  Sparkles,
  Coins,
  Gem,
  Sword,
  ShoppingCart,
  Backpack,
  Wand2,
  Gift,
  Clock3,
  Star,
  ScrollText,
  RotateCw,
} from "lucide-react";

const PRICE_MULTIPLIER = 1.4;
const EXPLORE_TIME_MULTIPLIER = 2.5;
const MAX_LEVEL = 1000;
const SAVE_KEY = "devil-fruit-collector-save-v1";
const SHOP_RESET_INTERVAL_MS = 60 * 60 * 1000;
const TRAVELER_DISCOUNT_INTERVAL_MS = 30 * 60 * 1000;
const DAILY_RESET_INTERVAL_MS = 24 * 60 * 60 * 1000;
const WEEKLY_RESET_INTERVAL_MS = 7 * 24 * 60 * 60 * 1000;
const MONSTER_FIGHT_COOLDOWN_MS = 5 * 60 * 1000;

const baseFruits = [
  { id: "rocket", name: "Rocket", tier: "Thường", power: 50, shopPrice: 50, awakenable: false },
  { id: "bomb", name: "Bom", tier: "Thường", power: 110, shopPrice: 100, awakenable: false },
  { id: "smoke", name: "Khói", tier: "Thường", power: 150, shopPrice: 140, awakenable: false },
  { id: "spring", name: "Lò xo", tier: "Thường", power: 190, shopPrice: 180, awakenable: false },
  { id: "eagle", name: "Đại bàng", tier: "Thường", power: 300, shopPrice: 280, awakenable: false },

  { id: "fire", name: "Lửa", tier: "Phổ biến", power: 1100, shopPrice: 2000, awakenable: true },
  { id: "ice", name: "Băng", tier: "Phổ biến", power: 1300, shopPrice: 2400, awakenable: false },
  { id: "sand", name: "Cát", tier: "Phổ biến", power: 1600, shopPrice: 3000, awakenable: false },
  { id: "dark", name: "Bóng tối", tier: "Phổ biến", power: 2000, shopPrice: 3800, awakenable: true },
  { id: "diamond", name: "Kim cương", tier: "Phổ biến", power: 2350, shopPrice: 4400, awakenable: false },
  { id: "rubber", name: "Cao su", tier: "Phổ biến", power: 2500, shopPrice: 4700, awakenable: false },
  { id: "light", name: "Ánh sáng", tier: "Phổ biến", power: 2900, shopPrice: 5400, awakenable: true },
  { id: "barrier", name: "Rào cản", tier: "Phổ biến", power: 3000, shopPrice: 5600, awakenable: false },
  { id: "magma", name: "Magma", tier: "Phổ biến", power: 3700, shopPrice: 7000, awakenable: false },

  { id: "quake", name: "Rung chấn", tier: "Hiếm", power: 6000, shopPrice: 30000, awakenable: false },
  { id: "buddha", name: "Phật tổ", tier: "Hiếm", power: 6800, shopPrice: 34000, awakenable: false },
  { id: "love", name: "Tình yêu", tier: "Hiếm", power: 7100, shopPrice: 36000, awakenable: false },
  { id: "spider", name: "Spider", tier: "Hiếm", power: 7250, shopPrice: 37000, awakenable: false },
  { id: "sound", name: "Âm thanh", tier: "Hiếm", power: 7600, shopPrice: 39000, awakenable: true },
  { id: "phoenix", name: "Phượng hoàng", tier: "Hiếm", power: 8000, shopPrice: 42000, awakenable: false },
  { id: "water", name: "Nước", tier: "Hiếm", power: 8500, shopPrice: null, awakenable: false, shopDisabled: true, eventOnly: true },

  { id: "portal", name: "Cổng dịch chuyển", tier: "Siêu hiếm", power: 14000, shopPrice: 80000, awakenable: false },
  { id: "thunder", name: "Sấm sét", tier: "Siêu hiếm", power: 17500, shopPrice: 100000, awakenable: false },
  { id: "pain", name: "Nỗi đau", tier: "Siêu hiếm", power: 20000, shopPrice: 115000, awakenable: false },
  { id: "frost", name: "Băng tuyết", tier: "Siêu hiếm", power: 26300, shopPrice: 150000, awakenable: false },

  { id: "control", name: "Điều khiển", tier: "Huyền thoại", power: 53000, shopPrice: 180000, awakenable: false },
  { id: "soul", name: "Linh hồn", tier: "Huyền thoại", power: 72000, shopPrice: 250000, awakenable: false },

  { id: "dragon", name: "Rồng", tier: "Đẳng cấp", power: 150000, shopPrice: 500000, awakenable: true },
];

const fruits = baseFruits.map((fruit) => ({
  ...fruit,
  shopPrice: fruit.shopPrice == null ? null : Math.floor(fruit.shopPrice * PRICE_MULTIPLIER),
}));

const tierColors = {
  "Thường": "bg-slate-100 text-slate-700",
  "Phổ biến": "bg-emerald-100 text-emerald-700",
  "Hiếm": "bg-sky-100 text-sky-700",
  "Siêu hiếm": "bg-violet-100 text-violet-700",
  "Huyền thoại": "bg-amber-100 text-amber-700",
  "Đẳng cấp": "bg-rose-100 text-rose-700",
};

const tierOrder = ["Thường", "Phổ biến", "Hiếm", "Siêu hiếm", "Huyền thoại", "Đẳng cấp"];

const baseRates = {
  "Thường": 48,
  "Phổ biến": 32,
  "Hiếm": 13,
  "Siêu hiếm": 5,
  "Huyền thoại": 1.7,
  "Đẳng cấp": 0.3,
};

const baseExploreChapters = [
  { id: 1, name: "Chương 1", minLevel: 1, maxLevel: 50, minSec: 45, maxSec: 90, goldMin: 100, goldMax: 800, exp: 18, diamondChance: 0.15, diamondReward: 3 },
  { id: 2, name: "Chương 2", minLevel: 51, maxLevel: 110, minSec: 50, maxSec: 105, goldMin: 500, goldMax: 1500, exp: 35, diamondChance: 0.18, diamondReward: 5 },
  { id: 3, name: "Chương 3", minLevel: 111, maxLevel: 190, minSec: 58, maxSec: 120, goldMin: 800, goldMax: 1650, exp: 60, diamondChance: 0.23, diamondReward: 8 },
  { id: 4, name: "Chương 4", minLevel: 191, maxLevel: 250, minSec: 65, maxSec: 135, goldMin: 1000, goldMax: 1800, exp: 95, diamondChance: 0.27, diamondReward: 10 },
  { id: 5, name: "Chương 5", minLevel: 251, maxLevel: 310, minSec: 75, maxSec: 150, goldMin: 1200, goldMax: 1900, exp: 145, diamondChance: 0.28, diamondReward: 11 },
  { id: 6, name: "Chương 6", minLevel: 311, maxLevel: 380, minSec: 82, maxSec: 165, goldMin: 1350, goldMax: 2100, exp: 190, diamondChance: 0.29, diamondReward: 14 },
  { id: 7, name: "Chương 7", minLevel: 381, maxLevel: 450, minSec: 90, maxSec: 180, goldMin: 1500, goldMax: 2200, exp: 235, diamondChance: 0.3, diamondReward: 15 },
  { id: 8, name: "Chương 8", minLevel: 451, maxLevel: 600, minSec: 100, maxSec: 195, goldMin: 1800, goldMax: 2500, exp: 280, diamondChance: 0.32, diamondReward: 17 },
  { id: 9, name: "Chương 9", minLevel: 601, maxLevel: 800, minSec: 110, maxSec: 210, goldMin: 2000, goldMax: 2700, exp: 330, diamondChance: 0.35, diamondReward: 20 },
  { id: 10, name: "Chương 10", minLevel: 801, maxLevel: 1000, minSec: 120, maxSec: 225, goldMin: 2200, goldMax: 3000, exp: 380, diamondChance: 0.35, diamondReward: 21 },
];

const exploreChapters = baseExploreChapters.map((chapter) => ({
  ...chapter,
  minSec: Math.floor(chapter.minSec * EXPLORE_TIME_MULTIPLIER),
  maxSec: Math.floor(chapter.maxSec * EXPLORE_TIME_MULTIPLIER),
}));

const marketItems = [
  { id: "luck5", name: "Thuốc may mắn 5%", price: 500, effect: 5 },
  { id: "luck10", name: "Thuốc may mắn 10%", price: 900, effect: 10 },
  { id: "luck20", name: "Thuốc may mắn 20%", price: 1500, effect: 20 },
  { id: "awakening", name: "Vé awakening", price: 3000, effect: "awake" },
];

const gamepassItems = [
  { id: "xp2_10m", name: "x2 EXP trong 10 phút", price: 200, multiplier: 2, durationMs: 10 * 60 * 1000 },
  { id: "xp2_20m", name: "x2 EXP trong 20 phút", price: 380, multiplier: 2, durationMs: 20 * 60 * 1000 },
  { id: "xp2_60m", name: "x2 EXP trong 1 tiếng", price: 1050, multiplier: 2, durationMs: 60 * 60 * 1000 },
  { id: "xp3_5m", name: "x3 EXP trong 5 phút", price: 500, multiplier: 3, durationMs: 5 * 60 * 1000 },
  { id: "xp3_10m", name: "x3 EXP trong 10 phút", price: 950, multiplier: 3, durationMs: 10 * 60 * 1000 },
];

const travelerDailyMissions = [
  { id: "d_lv_50", label: "Tăng thêm 50 Lv", type: "levelGain", target: 50, reward: 100 },
  { id: "d_lv_70", label: "Tăng thêm 70 Lv", type: "levelGain", target: 70, reward: 120 },
  { id: "d_lv_90", label: "Tăng thêm 90 Lv", type: "levelGain", target: 90, reward: 150 },
  { id: "d_lv_100", label: "Tăng thêm 100 Lv", type: "levelGain", target: 100, reward: 250 },
  { id: "d_spend_20000", label: "Tiêu 20.000 vàng", type: "goldSpent", target: 20000, reward: 100 },
  { id: "d_gain_1500", label: "Nhận 1.500 vàng", type: "goldEarned", target: 1500, reward: 200 },
  { id: "d_gain_5000", label: "Nhận 5.000 vàng", type: "goldEarned", target: 5000, reward: 250 },
  { id: "d_dia_5", label: "Nhận 5 kim cương", type: "diamondsEarned", target: 5, reward: 100 },
  { id: "d_dia_20", label: "Nhận 20 kim cương", type: "diamondsEarned", target: 20, reward: 400 },
];

const travelerWeeklyMissions = [
  { id: "w_lv_500", label: "Tăng thêm 500 Lv", type: "levelGain", target: 500, reward: 1000 },
  { id: "w_lv_700", label: "Tăng thêm 700 Lv", type: "levelGain", target: 700, reward: 2000 },
  { id: "w_gold_30000", label: "Nhận tổng 30.000 vàng", type: "goldEarned", target: 30000, reward: 1300 },
  { id: "w_gold_50000", label: "Nhận tổng 50.000 vàng", type: "goldEarned", target: 50000, reward: 1500 },
  { id: "w_dia_100", label: "Nhận tổng 100 kim cương", type: "diamondsEarned", target: 100, reward: 2000 },
  { id: "w_dia_200", label: "Nhận tổng 200 kim cương", type: "diamondsEarned", target: 200, reward: 2000 },
];

const travelerExchangeRewards = [
  { id: "ex_dia_5", cost: 400, label: "5 kim cương", kind: "diamonds", amount: 5 },
  { id: "ex_dia_13", cost: 700, label: "13 kim cương", kind: "diamonds", amount: 13 },
  { id: "ex_ticket_1", cost: 1300, label: "1 thẻ Lữ Khách", kind: "travelerTickets", amount: 1 },
  { id: "ex_ticket_2", cost: 2500, label: "2 thẻ Lữ Khách", kind: "travelerTickets", amount: 2 },
  { id: "ex_ticket_5", cost: 5500, label: "5 thẻ Lữ Khách", kind: "travelerTickets", amount: 5 },
  { id: "ex_gold_fire", cost: 15000, label: "Trái Lửa màu vàng óng ánh", kind: "fruit", fruitId: "fire_gold" },
  { id: "ex_red_ice", cost: 15000, label: "Trái Băng màu đỏ rực", kind: "fruit", fruitId: "ice_red" },
  { id: "ex_water", cost: 35000, label: "Trái Nước", kind: "fruit", fruitId: "water" },
  { id: "ex_blue_dragon", cost: 50000, label: "Trái Rồng màu xanh lam rực rỡ", kind: "fruit", fruitId: "dragon_blue" },
];

const travelerSpinRewards = [
  { id: "spin_dia_5", label: "5 kim cương", weight: 22, kind: "diamonds", amount: 5 },
  { id: "spin_dia_8", label: "8 kim cương", weight: 18, kind: "diamonds", amount: 8 },
  { id: "spin_dia_10", label: "10 kim cương", weight: 14, kind: "diamonds", amount: 10 },
  { id: "spin_pts_500", label: "500 điểm Lữ Khách", weight: 14, kind: "travelerPoints", amount: 500 },
  { id: "spin_pts_800", label: "800 điểm Lữ Khách", weight: 10, kind: "travelerPoints", amount: 800 },
  { id: "spin_pts_1000", label: "1000 điểm Lữ Khách", weight: 8, kind: "travelerPoints", amount: 1000 },
  { id: "spin_pts_1200", label: "1200 điểm Lữ Khách", weight: 5, kind: "travelerPoints", amount: 1200 },
  { id: "spin_random_card", label: "1 thẻ random", weight: 5.5, kind: "randomCards", amount: 1 },
  { id: "spin_awake", label: "1 vé awakening", weight: 2.7, kind: "awakeningTickets", amount: 1 },
  { id: "spin_fire_gold", label: "Trái Lửa màu vàng óng ánh", weight: 0.5, kind: "fruit", fruitId: "fire_gold" },
  { id: "spin_ice_red", label: "Trái Băng màu đỏ rực", weight: 0.25, kind: "fruit", fruitId: "ice_red" },
  { id: "spin_water", label: "Trái Nước", weight: 0.05, kind: "fruit", fruitId: "water" },
];

function randInt(min: number, max: number) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

function chance(prob: number) {
  return Math.random() < prob;
}

function weightedPick<T>(weights: { value: T; weight: number }[]): T {
  const total = weights.reduce((sum, item) => sum + item.weight, 0);
  let cursor = Math.random() * total;
  for (const item of weights) {
    cursor -= item.weight;
    if (cursor <= 0) return item.value;
  }
  return weights[weights.length - 1].value;
}

function getRates(luckPercent: number, hasSuperLucky: boolean) {
  const rates: Record<string, number> = { ...baseRates };
  const bonus = luckPercent + (hasSuperLucky ? 10 : 0);
  rates["Thường"] = Math.max(20, rates["Thường"] - bonus * 0.9);
  rates["Phổ biến"] = Math.max(18, rates["Phổ biến"] - bonus * 0.2);
  rates["Hiếm"] += bonus * 0.45;
  rates["Siêu hiếm"] += bonus * 0.35;
  rates["Huyền thoại"] += bonus * 0.22;
  rates["Đẳng cấp"] += bonus * 0.08;
  const total = Object.values(rates).reduce((a, b) => a + b, 0);
  const normalized: Record<string, number> = {};
  Object.keys(rates).forEach((key) => {
    normalized[key] = (rates[key] / total) * 100;
  });
  return normalized;
}

function pickFruit(luckPercent: number, hasSuperLucky: boolean) {
  const rates = getRates(luckPercent, hasSuperLucky);
  const chosenTier = weightedPick(tierOrder.map((tier) => ({ value: tier, weight: rates[tier] })));
  const pool = fruits.filter((fruit) => fruit.tier === chosenTier && !fruit.eventOnly);
  return weightedPick(pool.map((fruit, index) => ({ value: fruit, weight: index + 1 })));
}

function randomShopInventory() {
  const sellableFruits = fruits.filter((fruit) => !fruit.shopDisabled && fruit.shopPrice != null);
  const count = randInt(5, 9);
  const shuffled = [...sellableFruits].sort(() => Math.random() - 0.5).slice(0, count);
  return shuffled
    .map((fruit) => {
      const discount = chance(0.28) ? randInt(10, 50) : 0;
      return {
        ...fruit,
        discount,
        finalPrice: Math.floor((fruit.shopPrice! * (100 - discount)) / 100),
      };
    })
    .sort((a, b) => (a.shopPrice ?? 0) - (b.shopPrice ?? 0));
}

function getFruitName(fruit: any) {
  const prefix = fruit.specialColor ? `✨ ${fruit.specialColor} ` : fruit.awakened ? "✨ " : "";
  const suffix = fruit.awakened ? " (Awakened)" : "";
  return `${prefix}${fruit.name}${suffix}`;
}

function makeSpecialFruit(fruitId: string) {
  if (fruitId === "fire_gold") {
    const base = fruits.find((item) => item.id === "fire")!;
    return { ...base, id: `fire_gold_${Date.now()}`, uid: `fire_gold_${Date.now()}`, specialColor: "Lửa vàng óng ánh", power: 9800, awakened: false };
  }
  if (fruitId === "ice_red") {
    const base = fruits.find((item) => item.id === "ice")!;
    return { ...base, id: `ice_red_${Date.now()}`, uid: `ice_red_${Date.now()}`, specialColor: "Băng đỏ rực", power: 11200, awakened: false };
  }
  if (fruitId === "dragon_blue") {
    const base = fruits.find((item) => item.id === "dragon")!;
    return { ...base, id: `dragon_blue_${Date.now()}`, uid: `dragon_blue_${Date.now()}`, specialColor: "Rồng xanh lam rực rỡ", power: 195000, awakened: false };
  }
  const base = fruits.find((item) => item.id === fruitId)!;
  return { ...base, uid: `${fruitId}_${Date.now()}`, awakened: false };
}

function xpNeeded(level: number) {
  if (level <= 40) return 18 + level * 4;
  if (level <= 100) return 260 + (level - 40) * 24;
  if (level <= 500) return 1700 + (level - 100) * 42;
  return 18500 + (level - 500) * 65;
}

function alignFutureTime(intervalMs: number, now = Date.now()) {
  return Math.floor(now / intervalMs) * intervalMs + intervalMs;
}

function getHighestChapterForLevel(level: number) {
  const unlocked = exploreChapters.filter((chapter) => level >= chapter.minLevel);
  return unlocked.length > 0 ? unlocked[unlocked.length - 1].id : 1;
}

function createInitialSave(now = Date.now()) {
  return {
    gold: 1200,
    diamonds: 850,
    level: 1,
    xp: 0,
    selectedChapter: 1,
    highestUnlockedChapter: 1,
    currentFruit: null,
    inventory: [] as any[],
    superLuckyCards: 0,
    randomCards: 0,
    luckBonus: 0,
    awakeningTickets: 0,
    travelerPoints: 0,
    travelerTickets: 0,
    gachaResult: null as any,
    travelerSpinResult: null as string | null,
    logs: ["Sự kiện Lữ Khách Tham Gia đã bắt đầu!"],
    shopStock: randomShopInventory(),
    shopRefreshAt: alignFutureTime(SHOP_RESET_INTERVAL_MS, now),
    travelerDiscountAt: alignFutureTime(TRAVELER_DISCOUNT_INTERVAL_MS, now),
    dailyClaimed: [] as string[],
    weeklyClaimed: [] as string[],
    missionStats: { goldEarned: 0, goldSpent: 0, diamondsEarned: 0 },
    baselineLevel: 1,
    dailyResetAt: alignFutureTime(DAILY_RESET_INTERVAL_MS, now),
    weeklyResetAt: alignFutureTime(WEEKLY_RESET_INTERVAL_MS, now),
    expBoostMultiplier: 1,
    expBoostEndsAt: 0,
    gamepassRandomBundleVisible: chance(0.3),
    nextMonsterFightAt: 0,
  };
}

function loadSavedState() {
  if (typeof window === "undefined") return createInitialSave();
  try {
    const raw = window.localStorage.getItem(SAVE_KEY);
    if (!raw) return createInitialSave();
    const parsed = JSON.parse(raw);
    const initial = createInitialSave();
    return {
      ...initial,
      ...parsed,
      inventory: Array.isArray(parsed.inventory) ? parsed.inventory : initial.inventory,
      logs: Array.isArray(parsed.logs) && parsed.logs.length > 0 ? parsed.logs : initial.logs,
      shopStock: Array.isArray(parsed.shopStock) && parsed.shopStock.length > 0 ? parsed.shopStock : initial.shopStock,
      dailyClaimed: Array.isArray(parsed.dailyClaimed) ? parsed.dailyClaimed : initial.dailyClaimed,
      weeklyClaimed: Array.isArray(parsed.weeklyClaimed) ? parsed.weeklyClaimed : initial.weeklyClaimed,
      missionStats: {
        ...initial.missionStats,
        ...(parsed.missionStats ?? {}),
      },
    };
  } catch (error) {
    console.error("Failed to load save", error);
    return createInitialSave();
  }
}

function saveGameState(state: any) {
  if (typeof window === "undefined") return;
  try {
    window.localStorage.setItem(SAVE_KEY, JSON.stringify(state));
  } catch (error) {
    console.error("Failed to save state", error);
  }
}

function runSelfTests() {
  console.assert(xpNeeded(1) < xpNeeded(40), "xpNeeded should increase early game");
  console.assert(xpNeeded(100) < xpNeeded(500), "xpNeeded should increase later game");
  const rates = getRates(15, true);
  const totalRate = Object.values(rates).reduce((sum, value) => sum + value, 0);
  console.assert(Math.abs(totalRate - 100) < 0.001, "gacha rates should normalize to 100%");
  const shop = randomShopInventory();
  console.assert(shop.length >= 5 && shop.length <= 9, "shop inventory should have 5-9 fruits");
  console.assert(shop.every((item) => item.finalPrice > 0), "shop prices should stay positive");
  console.assert(!shop.some((item) => item.name === "Nước"), "event fruit water should not be sold in shop");
  console.assert(exploreChapters.length === 10, "explore chapters should have 10 stages");
  console.assert(exploreChapters[0].diamondReward === 3, "chapter 1 diamond reward should be fixed");
  console.assert(exploreChapters[9].maxLevel === 1000, "chapter 10 max level should be 1000");
  console.assert(getHighestChapterForLevel(58) === 2, "level 58 should only unlock chapter 2");
}

runSelfTests();

function applyTravelerDiscount(stock: any[]) {
  const chosen = [...stock].sort(() => Math.random() - 0.5).slice(0, 3).map((item) => item.id);
  return stock.map((item) => {
    if (!chosen.includes(item.id)) return item;
    const eventDiscount = randInt(5, 30);
    const bestDiscount = Math.max(item.discount ?? 0, eventDiscount);
    return {
      ...item,
      discount: bestDiscount,
      finalPrice: Math.floor(((item.shopPrice ?? item.finalPrice) * (100 - bestDiscount)) / 100),
    };
  });
}

function MissionCard({ mission, progress, onClaim }: { mission: any; progress: number; onClaim: (m: any) => void }) {
  const percent = Math.min(100, (progress / mission.target) * 100);
  return (
    <div className="rounded-2xl border p-4">
      <div className="flex items-start justify-between gap-3">
        <div>
          <div className="font-semibold">{mission.label}</div>
          <div className="mt-1 text-sm text-slate-500">Thưởng: {mission.reward.toLocaleString()} điểm Lữ Khách</div>
        </div>
        {mission.claimed ? (
          <Badge className="bg-emerald-100 text-emerald-700">Đã nhận</Badge>
        ) : progress >= mission.target ? (
          <Button size="sm" className="rounded-2xl" onClick={() => onClaim(mission)}>Nhận</Button>
        ) : (
          <Badge variant="outline">{progress.toLocaleString()}/{mission.target.toLocaleString()}</Badge>
        )}
      </div>
      <div className="mt-3">
        <Progress value={percent} className="h-2" />
      </div>
    </div>
  );
}

function Stat({ icon, label, value }: { icon: React.ReactNode; label: string; value: string }) {
  return (
    <div className="rounded-2xl bg-slate-50 p-4">
      <div className="flex items-center gap-2 text-sm text-slate-500">{icon}{label}</div>
      <div className="mt-1 text-xl font-bold">{value}</div>
    </div>
  );
}

function MiniInfo({ label, value }: { label: string; value: string }) {
  return (
    <div className="rounded-2xl border bg-white p-3 text-sm">
      <div className="text-slate-500">{label}</div>
      <div className="font-semibold">{value}</div>
    </div>
  );
}

export default function App() {
  const initialSave = useMemo(() => loadSavedState(), []);

  const [gold, setGold] = useState(initialSave.gold);
  const [diamonds, setDiamonds] = useState(initialSave.diamonds);
  const [level, setLevel] = useState(initialSave.level);
  const [xp, setXp] = useState(initialSave.xp);
  const [selectedChapter, setSelectedChapter] = useState(initialSave.selectedChapter);
  const [highestUnlockedChapter, setHighestUnlockedChapter] = useState(initialSave.highestUnlockedChapter);
  const [currentFruit, setCurrentFruit] = useState<any>(initialSave.currentFruit);
  const [inventory, setInventory] = useState<any[]>(initialSave.inventory);
  const [superLuckyCards, setSuperLuckyCards] = useState(initialSave.superLuckyCards);
  const [randomCards, setRandomCards] = useState(initialSave.randomCards);
  const [luckBonus, setLuckBonus] = useState(initialSave.luckBonus);
  const [awakeningTickets, setAwakeningTickets] = useState(initialSave.awakeningTickets);
  const [travelerPoints, setTravelerPoints] = useState(initialSave.travelerPoints);
  const [travelerTickets, setTravelerTickets] = useState(initialSave.travelerTickets);
  const [gachaResult, setGachaResult] = useState<any>(initialSave.gachaResult);
  const [travelerSpinResult, setTravelerSpinResult] = useState<string | null>(initialSave.travelerSpinResult);
  const [logs, setLogs] = useState<string[]>(initialSave.logs);
  const [shopStock, setShopStock] = useState<any[]>(initialSave.shopStock);
  const [shopRefreshAt, setShopRefreshAt] = useState(initialSave.shopRefreshAt);
  const [travelerDiscountAt, setTravelerDiscountAt] = useState(initialSave.travelerDiscountAt);
  const [exploring, setExploring] = useState(false);
  const [exploreProgress, setExploreProgress] = useState(0);
  const [awakeningLoading, setAwakeningLoading] = useState(false);
  const [dailyClaimed, setDailyClaimed] = useState<string[]>(initialSave.dailyClaimed);
  const [weeklyClaimed, setWeeklyClaimed] = useState<string[]>(initialSave.weeklyClaimed);
  const [dailyResetAt, setDailyResetAt] = useState(initialSave.dailyResetAt);
  const [weeklyResetAt, setWeeklyResetAt] = useState(initialSave.weeklyResetAt);
  const [expBoostMultiplier, setExpBoostMultiplier] = useState(initialSave.expBoostMultiplier ?? 1);
  const [expBoostEndsAt, setExpBoostEndsAt] = useState(initialSave.expBoostEndsAt ?? 0);
  const [gamepassRandomBundleVisible, setGamepassRandomBundleVisible] = useState(initialSave.gamepassRandomBundleVisible ?? chance(0.3));
  const [nextMonsterFightAt, setNextMonsterFightAt] = useState(initialSave.nextMonsterFightAt ?? 0);

  const baselineLevelRef = useRef(initialSave.baselineLevel ?? initialSave.level ?? 1);
  const missionStatsRef = useRef(initialSave.missionStats ?? { goldEarned: 0, goldSpent: 0, diamondsEarned: 0 });

  const maxChapterByLevel = useMemo(() => getHighestChapterForLevel(level), [level]);
  const effectiveUnlockedChapter = Math.min(highestUnlockedChapter, maxChapterByLevel);
  const chapter = exploreChapters.find((item) => item.id === selectedChapter) ?? exploreChapters[0];
  const activeFruitPower = currentFruit ? currentFruit.power * (currentFruit.awakened ? 1.8 : 1) : 10;

  const gachaCost = useMemo(() => {
    if (level < 40) return null;
    return Math.floor(1500 * Math.pow(1.012, Math.max(0, level - 40)) + Math.max(0, level - 40) * 120);
  }, [level]);

  const rates = useMemo(() => getRates(luckBonus, superLuckyCards > 0), [luckBonus, superLuckyCards]);

  const missionProgress = useMemo(
    () => ({
      levelGain: Math.max(0, level - baselineLevelRef.current),
      goldEarned: missionStatsRef.current.goldEarned,
      goldSpent: missionStatsRef.current.goldSpent,
      diamondsEarned: missionStatsRef.current.diamondsEarned,
    }),
    [level, gold, diamonds, travelerPoints, travelerTickets, inventory.length],
  );

  const currentExpBoostRemaining = Math.max(0, expBoostEndsAt - Date.now());
  const expBoostActive = currentExpBoostRemaining > 0;
  const monsterFightCooldownRemaining = Math.max(0, nextMonsterFightAt - Date.now());
  const canFightMonster = monsterFightCooldownRemaining <= 0;

  useEffect(() => {
    if (highestUnlockedChapter !== effectiveUnlockedChapter) {
      setHighestUnlockedChapter(effectiveUnlockedChapter);
    }
    if (selectedChapter > effectiveUnlockedChapter) {
      setSelectedChapter(effectiveUnlockedChapter);
    }
  }, [effectiveUnlockedChapter, highestUnlockedChapter, selectedChapter]);

  useEffect(() => {
    const timer = setInterval(() => {
      const now = Date.now();

      if (now >= shopRefreshAt) {
        setShopStock(randomShopInventory());
        setShopRefreshAt(alignFutureTime(SHOP_RESET_INTERVAL_MS, now));
        setGamepassRandomBundleVisible(chance(0.3));
        setLogs((prev) => ["Shop đã làm mới hàng hóa theo giờ thực.", ...prev].slice(0, 16));
      }

      if (now >= travelerDiscountAt) {
        setShopStock((prev) => applyTravelerDiscount(prev));
        setTravelerDiscountAt(alignFutureTime(TRAVELER_DISCOUNT_INTERVAL_MS, now));
        setLogs((prev) => ["🎉 Lữ Khách đã giảm giá 3 món trong shop theo mốc 30 phút thực!", ...prev].slice(0, 16));
      }

      if (now >= dailyResetAt) {
        setDailyClaimed([]);
        missionStatsRef.current = { ...missionStatsRef.current, goldEarned: 0, goldSpent: 0, diamondsEarned: 0 };
        baselineLevelRef.current = level;
        setDailyResetAt(alignFutureTime(DAILY_RESET_INTERVAL_MS, now));
        setLogs((prev) => ["📅 Nhiệm vụ ngày đã reset theo giờ thực.", ...prev].slice(0, 16));
      }

      if (now >= weeklyResetAt) {
        setWeeklyClaimed([]);
        missionStatsRef.current = { goldEarned: 0, goldSpent: 0, diamondsEarned: 0 };
        baselineLevelRef.current = level;
        setWeeklyResetAt(alignFutureTime(WEEKLY_RESET_INTERVAL_MS, now));
        setLogs((prev) => ["🗓️ Nhiệm vụ tuần đã reset theo giờ thực.", ...prev].slice(0, 16));
      }

      if (expBoostEndsAt > 0 && now >= expBoostEndsAt) {
        setExpBoostMultiplier(1);
        setExpBoostEndsAt(0);
        setLogs((prev) => ["⏳ Buff EXP đã hết hiệu lực.", ...prev].slice(0, 16));
      }
    }, 1000);

    return () => clearInterval(timer);
  }, [shopRefreshAt, travelerDiscountAt, dailyResetAt, weeklyResetAt, level, expBoostEndsAt]);

  useEffect(() => {
    saveGameState({
      gold,
      diamonds,
      level,
      xp,
      selectedChapter,
      highestUnlockedChapter,
      currentFruit,
      inventory,
      superLuckyCards,
      randomCards,
      luckBonus,
      awakeningTickets,
      travelerPoints,
      travelerTickets,
      gachaResult,
      travelerSpinResult,
      logs,
      shopStock,
      shopRefreshAt,
      travelerDiscountAt,
      dailyClaimed,
      weeklyClaimed,
      dailyResetAt,
      weeklyResetAt,
      expBoostMultiplier,
      expBoostEndsAt,
      gamepassRandomBundleVisible,
      nextMonsterFightAt,
      missionStats: missionStatsRef.current,
      baselineLevel: baselineLevelRef.current,
    });
  }, [gold, diamonds, level, xp, selectedChapter, highestUnlockedChapter, currentFruit, inventory, superLuckyCards, randomCards, luckBonus, awakeningTickets, travelerPoints, travelerTickets, gachaResult, travelerSpinResult, logs, shopStock, shopRefreshAt, travelerDiscountAt, dailyClaimed, weeklyClaimed, dailyResetAt, weeklyResetAt, expBoostMultiplier, expBoostEndsAt, gamepassRandomBundleVisible, nextMonsterFightAt]);

  const addLog = (text: string) => setLogs((prev) => [text, ...prev].slice(0, 16));

  const recordGoldEarned = (amount: number) => {
    if (amount <= 0) return;
    missionStatsRef.current.goldEarned += amount;
  };

  const recordGoldSpent = (amount: number) => {
    if (amount <= 0) return;
    missionStatsRef.current.goldSpent += amount;
  };

  const recordDiamondsEarned = (amount: number) => {
    if (amount <= 0) return;
    missionStatsRef.current.diamondsEarned += amount;
  };

  const claimMission = (mission: any, scope: "daily" | "weekly") => {
    setTravelerPoints((prev: number) => prev + mission.reward);
    addLog(`📜 Hoàn thành nhiệm vụ ${mission.label}, nhận ${mission.reward.toLocaleString()} điểm Lữ Khách.`);
    if (scope === "daily") setDailyClaimed((prev) => [...prev, mission.id]);
    if (scope === "weekly") setWeeklyClaimed((prev) => [...prev, mission.id]);
  };

  const applyXp = (amount: number) => {
    const boostedAmount = Math.floor(amount * expBoostMultiplier);
    setXp((prevXp: number) => {
      let newXp = prevXp + boostedAmount;
      let newLevel = level;
      let needed = xpNeeded(newLevel);
      const levelLogs: string[] = [];
      while (newXp >= needed && newLevel < MAX_LEVEL) {
        newXp -= needed;
        newLevel += 1;
        needed = xpNeeded(newLevel);
        levelLogs.push(`🎉 Lên cấp ${newLevel}!`);
      }
      if (newLevel >= MAX_LEVEL) {
        newLevel = MAX_LEVEL;
        newXp = 0;
      }
      if (newLevel !== level) {
        setLevel(newLevel);
        if (levelLogs.length > 0) {
          setLogs((prev) => [...levelLogs.reverse(), ...prev].slice(0, 16));
        }
      }
      return newXp;
    });
    if (expBoostMultiplier > 1) {
      addLog(`⚡ Buff EXP x${expBoostMultiplier} đã tăng EXP từ ${amount} lên ${boostedAmount}.`);
    }
  };

  const handleExplore = () => {
    if (exploring) return;
    if (selectedChapter > effectiveUnlockedChapter) {
      addLog("🔒 Bạn cần hoàn thành chương trước để mở chương này.");
      return;
    }
    if (level < chapter.minLevel || level > chapter.maxLevel) {
      addLog(`🔒 ${chapter.name} chỉ dành cho level ${chapter.minLevel}-${chapter.maxLevel}.`);
      return;
    }

    const durationSec = randInt(chapter.minSec, chapter.maxSec);
    const simulatedMs = Math.max(2400, Math.min(12000, durationSec * 55));
    setExploring(true);
    setExploreProgress(0);

    const started = Date.now();
    const interval = setInterval(() => {
      const progress = Math.min(100, ((Date.now() - started) / simulatedMs) * 100);
      setExploreProgress(progress);
    }, 80);

    setTimeout(() => {
      clearInterval(interval);
      setExploreProgress(100);
      setExploring(false);

      const goldGain = randInt(chapter.goldMin, chapter.goldMax);
      setGold((prev: number) => prev + goldGain);
      recordGoldEarned(goldGain);
      applyXp(chapter.exp + randInt(chapter.exp, chapter.exp * 3));
      addLog(`🧭 ${chapter.name}: nhận ${goldGain.toLocaleString()} vàng.`);

      if (chance(0.05)) {
        setSuperLuckyCards((prev: number) => prev + 1);
        addLog("🍀 Nhận được Thẻ siêu may mắn!");
      }

      if (chance(0.1)) {
        setRandomCards((prev: number) => prev + 1);
        addLog("🎴 Nhận được thẻ random!");
      }

      if (chance(chapter.diamondChance)) {
        const diamondsFound = chapter.diamondReward;
        setDiamonds((prev: number) => prev + diamondsFound);
        recordDiamondsEarned(diamondsFound);
        addLog(`💎 Tìm thấy ${diamondsFound} kim cương.`);
      }

      if (chance(0.18)) {
        const points = randInt(60, 180);
        setTravelerPoints((prev: number) => prev + points);
        addLog(`🌍 Gặp Lữ Khách bí ẩn, nhận ${points} điểm Lữ Khách.`);
      }

      if (chapter.id === effectiveUnlockedChapter && effectiveUnlockedChapter < exploreChapters.length) {
        const nextChapterId = chapter.id + 1;
        const nextChapter = exploreChapters[nextChapterId - 1];
        if (nextChapter && level >= nextChapter.minLevel) {
          setHighestUnlockedChapter(nextChapterId);
          addLog(`🗺️ Mở khóa ${nextChapter.name}.`);
        }
      }
    }, simulatedMs);
  };

  const handleFightMonster = () => {
    if (!currentFruit) {
      addLog("❌ Bạn phải có trái ác quỷ đang dùng mới có thể đánh quái.");
      return;
    }

    const now = Date.now();
    if (now < nextMonsterFightAt) {
      const secondsLeft = Math.ceil((nextMonsterFightAt - now) / 1000);
      const minsLeft = Math.floor(secondsLeft / 60);
      const secsLeft = secondsLeft % 60;
      addLog(`⏳ Bạn cần chờ ${minsLeft}:${String(secsLeft).padStart(2, "0")} nữa mới có thể đánh quái tiếp.`);
      return;
    }

    setNextMonsterFightAt(now + MONSTER_FIGHT_COOLDOWN_MS);

    const stagePowers = [0, 120, 2200, 7000, 22000, 60000, 85000, 120000, 155000, 190000, 230000];
    const stagePower = stagePowers[selectedChapter] ?? 230000;
    const effectivePower = activeFruitPower * (0.9 + level * 0.01);
    const successRate = Math.max(0.12, Math.min(0.95, effectivePower / (effectivePower + stagePower)));

    if (!chance(successRate)) {
      const lostGold = Math.min(gold, randInt(20, 120) * selectedChapter);
      setGold((prev: number) => Math.max(0, prev - lostGold));
      addLog(`💀 Đánh quái thất bại. Tỉ lệ thắng ${(successRate * 100).toFixed(0)}%. Mất ${lostGold.toLocaleString()} vàng.`);
      return;
    }

    const baseReward = Math.max(30, Math.floor(effectivePower / 8));
    const goldGain = randInt(baseReward, Math.floor(baseReward * 1.6));
    const xpGain = randInt(12, 28) + Math.floor(level * 0.5);
    setGold((prev: number) => prev + goldGain);
    recordGoldEarned(goldGain);
    applyXp(xpGain);
    addLog(`⚔️ Đánh quái thành công (${(successRate * 100).toFixed(0)}%), nhận ${goldGain.toLocaleString()} vàng và ${xpGain} EXP.`);
  };

  const doGacha = () => {
    if (level < 40) {
      addLog("🔒 Cần đạt level 40 mới có thể gacha.");
      return;
    }
    const cost = gachaCost ?? 0;
    if (gold < cost) {
      addLog(`❌ Không đủ vàng để gacha (cần ${cost.toLocaleString()} vàng).`);
      return;
    }
    setGold((prev: number) => prev - cost);
    recordGoldSpent(cost);
    const useSuperLucky = superLuckyCards > 0;
    if (useSuperLucky) setSuperLuckyCards((prev: number) => prev - 1);
    const result = pickFruit(luckBonus, useSuperLucky);
    setGachaResult({ ...result, awakened: false });
    addLog(`🎁 Gacha ra ${result.name} - ${result.tier}.`);
  };

  const eatFruit = () => {
    if (!gachaResult) return;
    setCurrentFruit({ ...gachaResult, source: "gacha" });
    setGachaResult(null);
    addLog(`🍽️ Đã ăn ${gachaResult.name}.`);
  };

  const storeFruit = () => {
    if (!gachaResult) return;
    setInventory((prev) => [...prev, { ...gachaResult, uid: `${gachaResult.id}-${Date.now()}` }]);
    setGachaResult(null);
    addLog(`🎒 Đã cất ${gachaResult.name} vào túi.`);
  };

  const tossFruit = () => {
    if (!gachaResult) return;
    addLog(`🗑️ Đã vứt ${gachaResult.name}.`);
    setGachaResult(null);
  };

  const useFromInventory = (item: any) => {
    setCurrentFruit({ ...item, source: "inventory" });
    setInventory((prev) => prev.filter((entry) => entry.uid !== item.uid));
    addLog(`⚡ Đã dùng ${item.name} từ túi.`);
  };

  const buyMarketItem = (item: any) => {
    if (diamonds < item.price) {
      addLog(`❌ Không đủ kim cương để mua ${item.name}.`);
      return;
    }
    setDiamonds((prev: number) => prev - item.price);
    if (item.id.startsWith("luck")) {
      setLuckBonus((prev: number) => prev + item.effect);
      addLog(`🧪 Dùng ${item.name}, tăng tỉ lệ gacha thêm ${item.effect}%.`);
      return;
    }
    if (item.id === "awakening") {
      setAwakeningTickets((prev: number) => prev + 1);
      addLog("🎟️ Mua thành công 1 Vé awakening.");
    }
  };

  const buyGamepass = (item: any) => {
    if (level < 500) {
      addLog("🔒 Cần level 500 mới có thể dùng GAMEPASS.");
      return;
    }
    if (diamonds < item.price) {
      addLog(`❌ Không đủ kim cương để mua ${item.name}.`);
      return;
    }
    setDiamonds((prev: number) => prev - item.price);
    const now = Date.now();
    const startFrom = expBoostEndsAt > now ? expBoostEndsAt : now;
    setExpBoostMultiplier(item.multiplier);
    setExpBoostEndsAt(startFrom + item.durationMs);
    addLog(`🚀 Kích hoạt ${item.name}.`);
  };

  const buyRandomBundle = () => {
    if (level < 500) {
      addLog("🔒 Cần level 500 mới có thể dùng GAMEPASS.");
      return;
    }
    if (!gamepassRandomBundleVisible) {
      addLog("❌ Gói 10 vé random hiện không xuất hiện trong shop.");
      return;
    }
    if (diamonds < 50) {
      addLog("❌ Không đủ kim cương để mua 10 vé random.");
      return;
    }
    setDiamonds((prev: number) => prev - 50);
    setRandomCards((prev: number) => prev + 10);
    setGamepassRandomBundleVisible(false);
    addLog("🎴 Mua thành công gói 10 vé random với giá 50 kim cương.");
  };

  const buyFruitFromShop = (fruit: any) => {
    if (gold < fruit.finalPrice) {
      addLog(`❌ Không đủ vàng mua ${fruit.name}.`);
      return;
    }
    setGold((prev: number) => prev - fruit.finalPrice);
    recordGoldSpent(fruit.finalPrice);
    setCurrentFruit({ ...fruit, awakened: false, source: "shop" });
    addLog(`🛒 Mua và ăn ngay ${fruit.name} với giá ${fruit.finalPrice.toLocaleString()} vàng.`);
  };

  const grantReward = (reward: any, sourceLabel: string) => {
    if (reward.kind === "diamonds") {
      setDiamonds((prev: number) => prev + reward.amount);
      recordDiamondsEarned(reward.amount);
      addLog(`🎁 ${sourceLabel}: nhận ${reward.amount} kim cương.`);
      return;
    }
    if (reward.kind === "travelerPoints") {
      setTravelerPoints((prev: number) => prev + reward.amount);
      addLog(`🎁 ${sourceLabel}: nhận ${reward.amount.toLocaleString()} điểm Lữ Khách.`);
      return;
    }
    if (reward.kind === "randomCards") {
      setRandomCards((prev: number) => prev + reward.amount);
      addLog(`🎁 ${sourceLabel}: nhận ${reward.amount} thẻ random.`);
      return;
    }
    if (reward.kind === "awakeningTickets") {
      setAwakeningTickets((prev: number) => prev + reward.amount);
      addLog(`🎁 ${sourceLabel}: nhận ${reward.amount} vé awakening.`);
      return;
    }
    if (reward.kind === "travelerTickets") {
      setTravelerTickets((prev: number) => prev + reward.amount);
      addLog(`🎁 ${sourceLabel}: nhận ${reward.amount} thẻ Lữ Khách.`);
      return;
    }
    if (reward.kind === "fruit") {
      const item = makeSpecialFruit(reward.fruitId);
      setInventory((prev) => [...prev, item]);
      addLog(`🎁 ${sourceLabel}: nhận ${getFruitName(item)} vào túi.`);
    }
  };

  const exchangeTravelerReward = (reward: any) => {
    if (travelerPoints < reward.cost) {
      addLog(`❌ Không đủ điểm Lữ Khách để đổi ${reward.label}.`);
      return;
    }
    setTravelerPoints((prev: number) => prev - reward.cost);
    grantReward(reward, "Đổi điểm Lữ Khách");
  };

  const spinTraveler = (count: number) => {
    const ticketCost = count === 6 ? 5 : 1;
    if (travelerTickets < ticketCost) {
      addLog("❌ Không đủ thẻ Lữ Khách để quay.");
      return;
    }
    setTravelerTickets((prev: number) => prev - ticketCost);
    const results: any[] = [];
    for (let i = 0; i < count; i += 1) {
      const reward = weightedPick(travelerSpinRewards.map((item) => ({ value: item, weight: item.weight })));
      results.push(reward);
      grantReward(reward, "Vòng quay Lữ Khách");
    }
    setTravelerSpinResult(results.map((item) => item.label).join(" • "));
  };

  const handleAwaken = () => {
    if (!currentFruit) {
      addLog("❌ Bạn chưa có trái đang dùng.");
      return;
    }
    if (!currentFruit.awakenable) {
      addLog("❌ Trái hiện tại không thể awakening.");
      return;
    }
    if (currentFruit.awakened) {
      addLog("✅ Trái này đã awakening rồi.");
      return;
    }
    if (awakeningTickets < 1) {
      addLog("❌ Bạn không có Vé awakening.");
      return;
    }
    const fruitName = currentFruit.name;
    setAwakeningTickets((prev: number) => prev - 1);
    setAwakeningLoading(true);
    addLog("⏳ Bắt đầu awakening... (mô phỏng 8 phút bằng 8 giây)");
    setTimeout(() => {
      const success = chance(0.3);
      if (success) {
        setCurrentFruit((prev: any) => (prev ? { ...prev, awakened: true } : prev));
        setDiamonds((prev: number) => prev + 250);
        setGold((prev: number) => prev + 50000);
        recordDiamondsEarned(250);
        recordGoldEarned(50000);
        addLog(`🌟 Awakening thành công cho ${fruitName}! Nhận 250 kim cương và 50.000 vàng.`);
      } else {
        addLog(`💥 Awakening thất bại cho ${fruitName}.`);
      }
      setAwakeningLoading(false);
    }, 8000);
  };

  const timeLeft = Math.max(0, shopRefreshAt - Date.now());
  const mins = Math.floor(timeLeft / 60000);
  const secs = Math.floor((timeLeft % 60000) / 1000);
  const travelerDiscountLeft = Math.max(0, travelerDiscountAt - Date.now());
  const travelerDiscountMins = Math.floor(travelerDiscountLeft / 60000);
  const travelerDiscountSecs = Math.floor((travelerDiscountLeft % 60000) / 1000);

  const fruitCollectionCount = new Set([
    ...(currentFruit ? [`${currentFruit.id}${currentFruit.awakened ? "-a" : ""}`] : []),
    ...inventory.map((item) => `${item.id}${item.awakened ? "-a" : ""}`),
  ]).size;

  return (
    <div className="min-h-screen bg-gradient-to-br from-slate-50 via-white to-amber-50 p-4 md:p-8">
      <div className="mx-auto max-w-7xl space-y-6">
        <motion.div initial={{ opacity: 0, y: 12 }} animate={{ opacity: 1, y: 0 }}>
          <Card className="rounded-3xl border-0 shadow-lg">
            <CardContent className="p-6 md:p-8">
              <div className="grid gap-4 md:grid-cols-3">
                <div>
                  <h1 className="text-3xl font-bold tracking-tight">Game Sưu Tầm Trái Ác Quỷ</h1>
                  <p className="mt-2 text-sm text-slate-600">Sự kiện Lữ Khách Tham Gia đã cập bến với nhiệm vụ, vòng quay, trái Nước mới và GAMEPASS.</p>
                </div>
                <div className="grid grid-cols-2 gap-3 md:col-span-2 md:grid-cols-5">
                  <Stat icon={<Coins className="h-4 w-4" />} label="Vàng" value={gold.toLocaleString()} />
                  <Stat icon={<Gem className="h-4 w-4" />} label="Kim cương" value={diamonds.toLocaleString()} />
                  <Stat icon={<Sparkles className="h-4 w-4" />} label="Level" value={String(level)} />
                  <Stat icon={<Star className="h-4 w-4" />} label="Điểm Lữ Khách" value={travelerPoints.toLocaleString()} />
                  <Stat icon={<Backpack className="h-4 w-4" />} label="Bộ sưu tập" value={String(fruitCollectionCount)} />
                </div>
              </div>
              <div className="mt-5 grid gap-3 md:grid-cols-5">
                <MiniInfo label="EXP" value={`${xp}/${xpNeeded(level)}`} />
                <MiniInfo label="Thẻ siêu may mắn" value={String(superLuckyCards)} />
                <MiniInfo label="Thẻ random" value={String(randomCards)} />
                <MiniInfo label="Vé awakening" value={String(awakeningTickets)} />
                <MiniInfo label="Thẻ Lữ Khách" value={String(travelerTickets)} />
              </div>
              <div className="mt-4">
                <Progress value={(xp / xpNeeded(level)) * 100} className="h-3" />
              </div>
              <div className="mt-3 text-xs text-slate-500">Game đã chuyển sang kiểu thời gian thực: shop, giảm giá, buff EXP và reset nhiệm vụ bám theo đồng hồ thật. Reload vẫn giữ đúng thời gian còn lại.</div>
            </CardContent>
          </Card>
        </motion.div>

        <div className="grid gap-6 xl:grid-cols-[1.2fr_0.8fr]">
          <Tabs defaultValue="explore" className="space-y-6">
            <TabsList className="grid w-full grid-cols-6 rounded-2xl">
              <TabsTrigger value="explore">Thám hiểm</TabsTrigger>
              <TabsTrigger value="gacha">Gacha</TabsTrigger>
              <TabsTrigger value="inventory">Túi đồ</TabsTrigger>
              <TabsTrigger value="shop">Shop</TabsTrigger>
              <TabsTrigger value="traveler">Lữ Khách</TabsTrigger>
              <TabsTrigger value="fruits">Bách khoa</TabsTrigger>
            </TabsList>

            <TabsContent value="explore">
              <Card className="rounded-3xl shadow-md">
                <CardHeader>
                  <CardTitle className="flex items-center gap-2"><Sword className="h-5 w-5" /> Thám hiểm & đánh quái</CardTitle>
                </CardHeader>
                <CardContent className="space-y-5">
                  <div className="grid gap-3 md:grid-cols-5">
                    {exploreChapters.map((item) => (
                      <button
                        key={item.id}
                        onClick={() => item.id <= effectiveUnlockedChapter && setSelectedChapter(item.id)}
                        className={`rounded-2xl border p-4 text-left transition ${selectedChapter === item.id ? "border-slate-900 bg-slate-900 text-white" : "bg-white hover:bg-slate-50"} ${item.id > effectiveUnlockedChapter ? "cursor-not-allowed opacity-50" : ""}`}
                      >
                        <div className="font-semibold">{item.name} {item.id > effectiveUnlockedChapter ? "🔒" : ""}</div>
                        <div className="mt-1 text-xs opacity-80">Lv {item.minLevel}-{item.maxLevel}</div>
                        <div className="text-xs opacity-80">{item.goldMin.toLocaleString()} - {item.goldMax.toLocaleString()} vàng</div>
                      </button>
                    ))}
                  </div>
                  <div className="rounded-2xl bg-slate-50 p-4">
                    <div className="grid gap-3 text-sm md:grid-cols-4">
                      <div>Level: <b>{chapter.minLevel} - {chapter.maxLevel}</b></div>
                      <div>Vàng: <b>{chapter.goldMin.toLocaleString()} - {chapter.goldMax.toLocaleString()}</b></div>
                      <div>Kim cương: <b>{Math.round(chapter.diamondChance * 100)}% · {chapter.diamondReward}</b></div>
                      <div>Lữ Khách: <b>18%</b> · May mắn <b>5%</b> · Random <b>10%</b></div>
                    </div>
                    {exploring && (
                      <div className="mt-4">
                        <Progress value={exploreProgress} className="h-3" />
                      </div>
                    )}
                  </div>
                  <div className="flex flex-wrap gap-3">
                    <Button onClick={handleExplore} disabled={exploring} className="rounded-2xl">Bắt đầu thám hiểm</Button>
                    <Button onClick={handleFightMonster} variant="secondary" className="rounded-2xl" disabled={!canFightMonster}>
                      {canFightMonster ? "Đánh quái kiếm vàng" : `Chờ ${Math.floor(monsterFightCooldownRemaining / 60000)}:${String(Math.floor((monsterFightCooldownRemaining % 60000) / 1000)).padStart(2, "0")}`}
                    </Button>
                  </div>
                </CardContent>
              </Card>
            </TabsContent>

            <TabsContent value="gacha">
              <Card className="rounded-3xl shadow-md">
                <CardHeader>
                  <CardTitle className="flex items-center gap-2"><Gift className="h-5 w-5" /> Người gacha</CardTitle>
                </CardHeader>
                <CardContent className="space-y-5">
                  <div className="grid gap-3 md:grid-cols-3">
                    <MiniInfo label="Phí gacha" value={level < 40 ? "Mở ở lv 40" : `${gachaCost?.toLocaleString()} vàng`} />
                    <MiniInfo label="Bonus may mắn hiện tại" value={`+${luckBonus}%`} />
                    <MiniInfo label="Tự dùng thẻ siêu may mắn" value={superLuckyCards > 0 ? "Có" : "Không"} />
                  </div>
                  <div className="grid gap-2 md:grid-cols-6">
                    {tierOrder.map((tier) => (
                      <div key={tier} className="rounded-2xl border bg-white p-3 text-center">
                        <div className="text-xs text-slate-500">{tier}</div>
                        <div className="font-semibold">{rates[tier].toFixed(2)}%</div>
                      </div>
                    ))}
                  </div>
                  <Button onClick={doGacha} className="rounded-2xl" disabled={level < 40}>Quay gacha</Button>
                  {gachaResult && (
                    <motion.div initial={{ opacity: 0, scale: 0.97 }} animate={{ opacity: 1, scale: 1 }} className="rounded-3xl border-2 border-dashed p-5">
                      <div className="flex items-center gap-3">
                        <Badge className={tierColors[gachaResult.tier]}>{gachaResult.tier}</Badge>
                        <div className="text-2xl font-bold">{gachaResult.name}</div>
                      </div>
                      <div className="mt-2 text-slate-600">Lực chiến: {gachaResult.power.toLocaleString()}</div>
                      <div className="mt-4 flex flex-wrap gap-3">
                        <Button onClick={eatFruit} className="rounded-2xl">Ăn</Button>
                        <Button onClick={storeFruit} variant="secondary" className="rounded-2xl">Cất vào túi</Button>
                        <Button onClick={tossFruit} variant="outline" className="rounded-2xl">Vứt</Button>
                      </div>
                    </motion.div>
                  )}
                </CardContent>
              </Card>
            </TabsContent>

            <TabsContent value="inventory">
              <div className="grid gap-6 lg:grid-cols-2">
                <Card className="rounded-3xl shadow-md">
                  <CardHeader><CardTitle>Trái đang dùng</CardTitle></CardHeader>
                  <CardContent>
                    {currentFruit ? (
                      <div className={`rounded-3xl p-5 ${currentFruit.awakened ? "bg-gradient-to-r from-amber-100 via-yellow-50 to-amber-100" : "bg-slate-50"}`}>
                        <div className="flex flex-wrap items-center gap-3">
                          <div className="text-2xl font-bold">{getFruitName(currentFruit)}</div>
                          <Badge className={tierColors[currentFruit.tier]}>{currentFruit.tier}</Badge>
                          {currentFruit.awakened && <Badge className="bg-yellow-200 text-yellow-900">Vàng lấp lánh</Badge>}
                        </div>
                        <div className="mt-3 space-y-1 text-sm text-slate-700">
                          <div>Lực chiến gốc: <b>{currentFruit.power.toLocaleString()}</b></div>
                          <div>Lực chiến hiện tại: <b>{Math.floor(activeFruitPower).toLocaleString()}</b></div>
                          <div>Nguồn: <b>{currentFruit.source}</b></div>
                        </div>
                        <div className="mt-4 flex flex-wrap gap-3">
                          <Button onClick={handleAwaken} disabled={awakeningLoading} className="rounded-2xl"><Wand2 className="mr-2 h-4 w-4" /> Awakening</Button>
                        </div>
                        <p className="mt-3 text-xs text-slate-500">Chỉ các trái: ánh sáng, bóng tối, lửa, âm thanh, rồng có thể awakening. Tỉ lệ thành công 30%. Thời gian awakening là 8 phút, trong bản demo mô phỏng bằng 8 giây.</p>
                      </div>
                    ) : (
                      <div className="rounded-2xl bg-slate-50 p-5 text-slate-500">Bạn chưa ăn trái nào.</div>
                    )}
                  </CardContent>
                </Card>
                <Card className="rounded-3xl shadow-md">
                  <CardHeader><CardTitle>Inventory</CardTitle></CardHeader>
                  <CardContent>
                    <div className="max-h-[520px] space-y-3 overflow-auto pr-1">
                      {inventory.length === 0 && <div className="rounded-2xl bg-slate-50 p-5 text-slate-500">Túi đang trống.</div>}
                      {inventory.map((item) => (
                        <div key={item.uid} className="rounded-2xl border p-4">
                          <div className="flex items-center justify-between gap-3">
                            <div>
                              <div className="font-semibold">{getFruitName(item)}</div>
                              <div className="text-sm text-slate-500">Lực chiến {item.power.toLocaleString()}</div>
                            </div>
                            <Badge className={tierColors[item.tier]}>{item.tier}</Badge>
                          </div>
                          <Button onClick={() => useFromInventory(item)} className="mt-3 rounded-2xl" size="sm">Dùng trái này</Button>
                        </div>
                      ))}
                    </div>
                  </CardContent>
                </Card>
              </div>
            </TabsContent>

            <TabsContent value="shop">
              <div className="space-y-4">
                <Card className="rounded-3xl shadow-md">
                  <CardContent className="p-4">
                    <div className="grid gap-3 md:grid-cols-4">
                      <MiniInfo label="Reset shop" value={`${String(mins).padStart(2, "0")}:${String(secs).padStart(2, "0")}`} />
                      <MiniInfo label="Giảm giá Lữ Khách" value={`${String(travelerDiscountMins).padStart(2, "0")}:${String(travelerDiscountSecs).padStart(2, "0")}`} />
                      <MiniInfo label="Buff EXP" value={expBoostActive ? `x${expBoostMultiplier}` : "Không có"} />
                      <MiniInfo label="GAMEPASS" value={level >= 500 ? "Đã mở" : "Mở ở lv 500"} />
                    </div>
                  </CardContent>
                </Card>

                <div className="grid gap-4 xl:grid-cols-[0.9fr_1.1fr]">
                  <div className="space-y-4">
                    <Card className="rounded-3xl shadow-md">
                      <CardHeader className="pb-3">
                        <CardTitle className="flex items-center gap-2 text-lg"><ShoppingCart className="h-5 w-5" /> Vật phẩm chợ</CardTitle>
                      </CardHeader>
                      <CardContent className="space-y-2">
                        {marketItems.map((item) => (
                          <div key={item.id} className="flex items-center justify-between rounded-2xl border p-3">
                            <div>
                              <div className="font-semibold">{item.name}</div>
                              <div className="text-xs text-slate-500">{item.id === "awakening" ? "Nâng cấp trái đủ điều kiện" : `+${item.effect}% may mắn`}</div>
                            </div>
                            <div className="flex items-center gap-2">
                              <div className="text-sm font-semibold">{item.price.toLocaleString()} 💎</div>
                              <Button size="sm" className="rounded-2xl" onClick={() => buyMarketItem(item)}>Mua</Button>
                            </div>
                          </div>
                        ))}
                      </CardContent>
                    </Card>

                    <Card className="rounded-3xl shadow-md">
                      <CardHeader className="pb-3">
                        <CardTitle className="flex items-center gap-2 text-lg"><Star className="h-5 w-5" /> GAMEPASS</CardTitle>
                      </CardHeader>
                      <CardContent className="space-y-2">
                        <div className="rounded-2xl bg-slate-50 p-3 text-xs text-slate-600">
                          {expBoostActive ? `Buff đang chạy x${expBoostMultiplier}, còn ${Math.ceil(currentExpBoostRemaining / 60000)} phút.` : "Mở ở level 500. Buff EXP chạy theo thời gian thực."}
                        </div>
                        {gamepassItems.map((item) => (
                          <div key={item.id} className="flex items-center justify-between rounded-2xl border p-3">
                            <div>
                              <div className="font-semibold">{item.name}</div>
                              <div className="text-xs text-slate-500">EXP x{item.multiplier}</div>
                            </div>
                            <div className="flex items-center gap-2">
                              <div className="text-sm font-semibold">{item.price.toLocaleString()} 💎</div>
                              <Button size="sm" className="rounded-2xl" onClick={() => buyGamepass(item)} disabled={level < 500}>Mua</Button>
                            </div>
                          </div>
                        ))}
                        <div className="flex items-center justify-between rounded-2xl border p-3">
                          <div>
                            <div className="font-semibold">10 vé random</div>
                            <div className="text-xs text-slate-500">30% xuất hiện khi shop reset</div>
                          </div>
                          <div className="flex items-center gap-2">
                            <div className="text-sm font-semibold">50 💎</div>
                            <Button size="sm" className="rounded-2xl" onClick={buyRandomBundle} disabled={level < 500 || !gamepassRandomBundleVisible}>{gamepassRandomBundleVisible ? "Mua" : "Ẩn"}</Button>
                          </div>
                        </div>
                      </CardContent>
                    </Card>
                  </div>

                  <Card className="rounded-3xl shadow-md">
                    <CardHeader className="pb-3">
                      <CardTitle className="flex items-center gap-2 text-lg"><Clock3 className="h-5 w-5" /> Trái ác quỷ đang bán</CardTitle>
                    </CardHeader>
                    <CardContent>
                      <div className="grid gap-3 md:grid-cols-2">
                        {shopStock.map((fruit) => (
                          <div key={fruit.id} className="rounded-2xl border p-3">
                            <div className="flex items-center justify-between gap-2">
                              <div>
                                <div className="font-semibold">{fruit.name}</div>
                                <div className="text-xs text-slate-500">{fruit.power.toLocaleString()} lực chiến</div>
                              </div>
                              <Badge className={tierColors[fruit.tier]}>{fruit.tier}</Badge>
                            </div>
                            <div className="mt-3 flex items-center justify-between gap-2">
                              <div className="text-sm">
                                {fruit.discount > 0 ? (
                                  <div>
                                    <span className="mr-2 line-through text-slate-400">{fruit.shopPrice?.toLocaleString()}</span>
                                    <span className="font-bold text-emerald-700">{fruit.finalPrice.toLocaleString()}</span>
                                    <span className="ml-2 text-xs text-rose-600">-{fruit.discount}%</span>
                                  </div>
                                ) : (
                                  <div className="font-bold">{fruit.finalPrice.toLocaleString()} vàng</div>
                                )}
                              </div>
                              <Button size="sm" className="rounded-2xl" onClick={() => buyFruitFromShop(fruit)}>Mua</Button>
                            </div>
                          </div>
                        ))}
                      </div>
                    </CardContent>
                  </Card>
                </div>
              </div>
            </TabsContent>

            <TabsContent value="traveler">
              <div className="space-y-6">
                <Card className="rounded-3xl shadow-md">
                  <CardHeader><CardTitle className="flex items-center gap-2"><ScrollText className="h-5 w-5" /> Sự kiện: Lữ Khách Tham Gia</CardTitle></CardHeader>
                  <CardContent>
                    <div className="grid gap-4 md:grid-cols-3">
                      <MiniInfo label="Điểm Lữ Khách" value={travelerPoints.toLocaleString()} />
                      <MiniInfo label="Thẻ Lữ Khách" value={travelerTickets.toLocaleString()} />
                      <MiniInfo label="Trái mới" value="Nước · 8.500 lực chiến" />
                    </div>
                  </CardContent>
                </Card>

                <div className="grid gap-6 xl:grid-cols-2">
                  <Card className="rounded-3xl shadow-md">
                    <CardHeader><CardTitle>Nhiệm vụ ngày</CardTitle></CardHeader>
                    <CardContent className="space-y-3">
                      {travelerDailyMissions.map((mission) => {
                        const progress = missionProgress[mission.type] ?? 0;
                        return <MissionCard key={mission.id} mission={{ ...mission, claimed: dailyClaimed.includes(mission.id) }} progress={progress} onClaim={(m) => claimMission(m, "daily")} />;
                      })}
                    </CardContent>
                  </Card>
                  <Card className="rounded-3xl shadow-md">
                    <CardHeader><CardTitle>Nhiệm vụ tuần</CardTitle></CardHeader>
                    <CardContent className="space-y-3">
                      {travelerWeeklyMissions.map((mission) => {
                        const progress = missionProgress[mission.type] ?? 0;
                        return <MissionCard key={mission.id} mission={{ ...mission, claimed: weeklyClaimed.includes(mission.id) }} progress={progress} onClaim={(m) => claimMission(m, "weekly")} />;
                      })}
                    </CardContent>
                  </Card>
                </div>

                <div className="grid gap-6 xl:grid-cols-2">
                  <Card className="rounded-3xl shadow-md">
                    <CardHeader><CardTitle>Đổi điểm Lữ Khách</CardTitle></CardHeader>
                    <CardContent className="space-y-3">
                      {travelerExchangeRewards.map((reward) => (
                        <div key={reward.id} className="flex items-center justify-between rounded-2xl border p-4">
                          <div>
                            <div className="font-semibold">{reward.label}</div>
                            <div className="text-sm text-slate-500">Giá đổi: {reward.cost.toLocaleString()} điểm</div>
                          </div>
                          <Button size="sm" className="rounded-2xl" onClick={() => exchangeTravelerReward(reward)}>Đổi</Button>
                        </div>
                      ))}
                    </CardContent>
                  </Card>
                  <Card className="rounded-3xl shadow-md">
                    <CardHeader><CardTitle className="flex items-center gap-2"><RotateCw className="h-5 w-5" /> Vòng quay Lữ Khách</CardTitle></CardHeader>
                    <CardContent className="space-y-4">
                      <div className="rounded-2xl bg-slate-50 p-4 text-sm text-slate-600">1 thẻ / 1 lần · 5 thẻ / 5 lần + 1 lần miễn phí</div>
                      <div className="flex flex-wrap gap-3">
                        <Button className="rounded-2xl" onClick={() => spinTraveler(1)}>Quay 1 lần</Button>
                        <Button className="rounded-2xl" variant="secondary" onClick={() => spinTraveler(6)}>Quay 5 + 1</Button>
                      </div>
                      {travelerSpinResult && <div className="rounded-2xl border border-dashed p-4 text-sm"><b>Kết quả gần nhất:</b> {travelerSpinResult}</div>}
                      <div className="grid gap-2 md:grid-cols-2">
                        {travelerSpinRewards.map((reward) => (
                          <div key={reward.id} className="rounded-2xl border p-3 text-sm">
                            <div className="font-medium">{reward.label}</div>
                            <div className="text-slate-500">Tỉ lệ: {reward.weight}% tương đối</div>
                          </div>
                        ))}
                      </div>
                    </CardContent>
                  </Card>
                </div>
              </div>
            </TabsContent>

            <TabsContent value="fruits">
              <Card className="rounded-3xl shadow-md">
                <CardHeader><CardTitle>Danh sách trái ác quỷ</CardTitle></CardHeader>
                <CardContent>
                  <div className="grid gap-3 md:grid-cols-2 xl:grid-cols-3">
                    {fruits.map((fruit) => (
                      <div key={fruit.id} className="rounded-2xl border p-4">
                        <div className="flex items-center justify-between gap-2">
                          <div className="font-semibold">{fruit.name}</div>
                          <Badge className={tierColors[fruit.tier]}>{fruit.tier}</Badge>
                        </div>
                        <div className="mt-2 text-sm text-slate-600">Lực chiến: <b>{fruit.power.toLocaleString()}</b></div>
                        <div className="text-sm text-slate-600">Giá shop: <b>{fruit.shopPrice == null ? "Chưa bán trong shop" : `${fruit.shopPrice.toLocaleString()} vàng`}</b></div>
                        <div className="mt-2 text-xs text-slate-500">{fruit.eventOnly ? "Trái sự kiện" : fruit.awakenable ? "Có thể awakening" : "Chưa thể awakening"}</div>
                      </div>
                    ))}
                  </div>
                </CardContent>
              </Card>
            </TabsContent>
          </Tabs>

          <div className="space-y-6">
            <Card className="rounded-3xl shadow-md">
              <CardHeader><CardTitle>Nhân vật</CardTitle></CardHeader>
              <CardContent>
                <div className="rounded-3xl bg-slate-50 p-5">
                  <div className="text-sm text-slate-500">Sức mạnh hiện tại</div>
                  <div className="mt-1 text-4xl font-bold">{Math.floor(activeFruitPower).toLocaleString()}</div>
                  <div className="mt-3 text-sm text-slate-600">Trái đang dùng: <b>{currentFruit ? getFruitName(currentFruit) : "Chưa có"}</b></div>
                </div>
              </CardContent>
            </Card>
            <Card className="rounded-3xl shadow-md">
              <CardHeader><CardTitle>Nhật ký hoạt động</CardTitle></CardHeader>
              <CardContent>
                <div className="space-y-3">
                  {logs.map((log, index) => (
                    <div key={index} className="rounded-2xl bg-slate-50 p-3 text-sm text-slate-700">{log}</div>
                  ))}
                </div>
              </CardContent>
            </Card>
            <Card className="rounded-3xl shadow-md">
              <CardHeader><CardTitle>Ghi chú luật game</CardTitle></CardHeader>
              <CardContent className="space-y-2 text-sm text-slate-600">
                <p>- Tất cả trái ác quỷ trong shop đã tăng giá 40%.</p>
                <p>- Thám hiểm đã được làm lại thành 10 chương với mốc vàng và kim cương riêng cho từng chương.</p>
                <p>- Tỉ lệ gặp Lữ Khách bí ẩn, thẻ siêu may mắn và thẻ random vẫn giữ nguyên như trước.</p>
                <p>- Trái Nước là trái hiếm mới, hiện chỉ nhận từ sự kiện Lữ Khách.</p>
                <p>- Mỗi 30 phút, Lữ Khách sẽ giảm giá 3 món trong shop tối đa 30%.</p>
                <p>- Thuốc may mắn cộng dồn vĩnh viễn trong bản demo này.</p>
                <p>- Shop 1 giờ, giảm giá 30 phút, nhiệm vụ ngày và tuần đều bám theo thời gian thực và không reset chỉ vì bạn thoát game.</p>
                <p>- GAMEPASS mở ở level 500, buff EXP hoạt động theo thời gian thực kể cả khi reload hoặc thoát game.</p>
                <p>- Giá gacha tăng theo level bằng công thức lũy tiến, càng lên cao càng tốn nhiều vàng.</p>
                <p>- Gói 10 vé random chỉ có 30% xuất hiện mỗi lần shop làm mới.</p>
                <p>- Đánh quái kiếm vàng có cooldown 5 phút mỗi lần và vẫn giữ khi reload game.</p>
                <p>- Mỗi chương thám hiểm hiện có khung level riêng, và level tối đa đã được chỉnh xuống 1000.</p>
              </CardContent>
            </Card>
          </div>
        </div>
      </div>
    </div>
  );
}
