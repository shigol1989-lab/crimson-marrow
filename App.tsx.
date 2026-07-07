// ============================================================================
// CRIMSON MARROW † WALK QUIETLY. SHARPEN DAILY.
// TEK PARÇA BÜTÜNLEŞİK SİBER GERİLLA KODU
// ============================================================================

import React, { useState, useMemo, useEffect, type ReactNode, useSyncExternalStore } from "react";
import { z } from "zod";
import { toast } from "sonner";
import { ChevronRight, Link2, Check } from "lucide-react";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { Outlet, Link, createRootRouteWithContext, useRouter, HeadContent, Scripts, createFileRoute, notFound } from "@tanstack/react-router";

// ----------------------------------------------------------------------------
// 1. VERİ MİMARİSİ & STORE (Eski: src/lib/blog-store.ts)
// ----------------------------------------------------------------------------

export type Comment = {
  id: string;
  postSlug: string;
  parentId: string | null;
  author: string;
  body: string;
  createdAt: string;
};

export type Category = "Laboratuvar" | "Zindan" | "Amnezi" | "Vicdan";
export const CATEGORIES: Category[] = ["Laboratuvar", "Zindan", "Amnezi", "Vicdan"];

export const CATEGORY_META: Record<Category, { blurb: string; symbol: string }> = {
  Laboratuvar: { blurb: "Deneyler, formüller, kuram taslakları.", symbol: "☣" },
  Zindan: { blurb: "Karanlık düşüncelerin hücreleri.", symbol: "☠" },
  Amnezi: { blurb: "Unutulmuş anıların kırıntıları.", symbol: "☿" },
  Vicdan: { blurb: "Ahlaki muhasebeler ve itiraflar.", symbol: "☨" },
};

export type Post = {
  slug: string;
  title: string;
  excerpt: string;
  body: string;
  author: string;
  category: Category;
  tags: string[];
  publishedAt: string;
  readingMinutes: number;
};

const POSTS_KEY = "cm_posts_v2";
const COMMENTS_KEY = "cm_comments_v2";
const FOLLOWERS_KEY = "cm_followers_v2";
const FOLLOWED_KEY = "cm_hasFollowed_v2";

const seedPosts: Post[] = [
  {
    slug: "kara-kilicin-agirligi",
    title: "Kara Kılıcın Ağırlığı",
    excerpt: "Hiçbir insanın taşıyamayacağı kadar ağır bir bıçak ve kolu düşmekten alıkoyan yemin üzerine.",
    body: "Salınmadan önce bir an vardır; kılıç dünyadan daha ağırdır.\n\nBu, demirin ağırlığı değildir. Bu, çelikte hapsolmuş her ismin, tutulmuş her sözün ağırlığıdır.\n\nBöyle bir silahı taşıyan savaşçı metal kaldırmaz. Kurtaramadıklarının hayaletlerini kaldırır. Ve her sabah, yeniden kaldırır.",
    author: "Otorite",
    category: "Vicdan",
    tags: ["felsefe", "çelik"],
    publishedAt: "2026-06-14T09:00:00Z",
    readingMinutes: 4
  },
  {
    slug: "tutulmanin-altinda-notlar",
    title: "Tutulmanın Altında Yazılmış Notlar",
    excerpt: "Güneşin karardığı ve göğün kutsal metin gibi kanadığı saatten bir saha günlüğü.",
    body: "Güneş üçüncü kez tutuldu.\n\nKöpekler çığlık atmayı kesmedi. Mürekkep kuyusunda dondu. Pusulam gerçek bir kuzey arıyormuş gibi ağır ağır döndü.\n\nBurada yazdıklarımı, sırt çantasını BCA’lar için yazıyorum. Tanıdık bir sesle konuşan hiçbir şeye güvenmeyin.",
    author: "Otorite",
    category: "Amnezi",
    tags: ["günlük", "tutulma"],
    publishedAt: "2026-05-02T09:00:00Z",
    readingMinutes: 6
  },
  {
    slug: "sahinler-ve-hirs",
    title: "Şahinler, Rüyalar ve Hırsın Bedeli",
    excerpt: "Bir şahini güneşe kadar kovalayan her adam iki şey öğrenir: güneş umursamaz, ve şahin asla onun değildi.",
    body: "Rüyalar bir insanın harcayabileceği en ucuz para, ödünç alabileceği en pahalı borçtur.\n\nHırs kötü değildir. Bedelsiz hırs kötüdür. Zirvenin bedava olduğunu söyleyen kimse tepede neyin beklediğini görmemiştir.",
    author: "Otorite",
    category: "Vicdan",
    tags: ["felsefe", "hırs"],
    publishedAt: "2026-05-20T09:00:00Z",
    readingMinutes: 5
  },
  {
    slug: "kapanmayan-damga",
    title: "Kapanmayan Damga",
    excerpt: "Her gece karanlığı sana çağıran bir işaretle yaşamak üzerine.",
    body: "Bir yanık olarak başladı. Şimdi bir işaret ateşi.\n\nGündüz sessizdir. Alacakaranlıkta ısınır. Tam karanlıkta bir çağrıdır ve dilden eski şeyler ona cevap verir.\n\nHayatta kalmanın bir hediye değil, disiplin olduğunu öğrendim.",
    author: "Otorite",
    category: "Zindan",
    tags: ["lanet"],
    publishedAt: "2025-11-11T09:00:00Z",
    readingMinutes: 3
  },
  {
    slug: "unutus-formulu",
    title: "Unutuşun Formülü",
    excerpt: "Bir ismi hatırlamaya bırakmak için gerekli maddeler ve prosedürler.",
    body: "İlk kural: onu bir kez daha söyleme.\n\nİkinci kural: söyleyeni dinleme.\n\nÜçüncü kural: eğer hatırlarsan, hatırladığını hatırlama.",
    author: "Otorite",
    category: "Laboratuvar",
    tags: ["deney", "hafıza"],
    publishedAt: "2025-09-03T09:00:00Z",
    readingMinutes: 4
  },
  {
    slug: "kuzey-ocaginin-demircileri",
    title: "Kuzey Ocağının Demircileri",
    excerpt: "Hâlâ ejder nefesinde ve nehir buzunda çelik tavlayan son yaşayan ocağa bir ziyaret.",
    body: "Az konuşurlar. Körükler onlar adına konuşur. Ocak, bir adamın sesini bir mevsimde alır ve ateş olarak geri verir.\n\nBir bıçağı kırk yedi kez katladıklarını izledim. Kırk yedi, savaşın aldığı demirci Berulf'un yaşadığı yıl sayısıdır.",
    author: "Otorite",
    category: "Laboratuvar",
    tags: ["zanaat", "seyahat"],
    publishedAt: "2025-08-22T09:00:00Z",
    readingMinutes: 7
  },
  {
    slug: "kayip-dosta-mektuplar",
    title: "Kayıp Bir Dosta Mektuplar",
    excerpt: "Ölülere yazmayı bırakamazsın. Sadece cevap beklemeyi bırakırsın.",
    body: "Hâlâ sana yazıyorum. Bilmiyorum. Aptalca.\n\nAma yazı senin için değil. Sessizliği henüz kabul etmemiş olan bendeki versiyon için. Ona cümle cümle hatırlatılması gerek.",
    author: "Otorite",
    category: "Amnezi",
    tags: ["mektuplar"],
    publishedAt: "2025-02-14T09:00:00Z",
    readingMinutes: 3
  },
  {
    slug: "berserker-zirhi",
    title: "Berserker Zırhı: Öfke Üzerine Bir Meditasyon",
    excerpt: "Giyilebilen bir öfke vardır. Ve seni giyen bir öfke.",
    body: "Öfke bir araçtır. Öfke bir borçtur. Zırhı giydiğinde daha güçlü olmam – yarını bugüne feda ederim. Her darbe, bir gün boşalacak bir keseden bir sikkedir.\n\nYine de giyerim. Bazı savaşlar tek başına bir adam tarafından kazanılmaz.",
    author: "Otorite",
    category: "Zindan",
    tags: ["felsefe", "lanet"],
    publishedAt: "2024-12-06T09:00:00Z",
    readingMinutes: 5
  }
];

const seedComments: Comment[] = [
  { id: "c1", postSlug: "kara-kilicin-agirligi", parentId: null, author: "Hacı", body: "Bu bana derinden dokundu. Her zanaatkâr bu ağırlığı bilir.", createdAt: "2026-06-15T12:00:00Z" },
  { id: "c2", postSlug: "kara-kilicin-agirligi", parentId: "c1", author: "Otorite", body: "Zanaat aynıdır. Kılıç sadece elime uyan metafordu.", createdAt: "2026-06-15T18:30:00Z" }
];

const isBrowser = () => typeof window !== "undefined";

function read<T>(key: string, fallback: T): T {
  if (!isBrowser()) return fallback;
  try {
    const raw = window.localStorage.getItem(key);
    return raw ? (JSON.parse(raw) as T) : fallback;
  } catch { return fallback; }
}

function write<T>(key: string, value: T) {
  if (!isBrowser()) return;
  window.localStorage.setItem(key, JSON.stringify(value));
}

let cachedPosts: Post[] | null = null;
let cachedComments: Comment[] | null = null;
let cachedArchive: Archive | null = null;
let cachedTagCounts: { tag: string; count: number }[] | null = null;
let cachedCategoryCounts: Record<Category, number> | null = null;

function ensureSeeded() {
  if (!isBrowser()) return;
  if (!window.localStorage.getItem(POSTS_KEY)) write(POSTS_KEY, seedPosts);
  if (!window.localStorage.getItem(COMMENTS_KEY)) write(COMMENTS_KEY, seedComments);
  if (!window.localStorage.getItem(FOLLOWERS_KEY)) write(FOLLOWERS_KEY, 1247);
}

function loadPosts(): Post[] {
  if (cachedPosts) return cachedPosts;
  ensureSeeded();
  const posts = read<Post[]>(POSTS_KEY, seedPosts);
  cachedPosts = [...posts].sort((a, b) => new Date(b.publishedAt).getTime() - new Date(a.publishedAt).getTime());
  return cachedPosts;
}

function loadComments(): Comment[] {
  if (cachedComments) return cachedComments;
  ensureSeeded();
  cachedComments = read<Comment[]>(COMMENTS_KEY, seedComments);
  return cachedComments;
}

const listeners = new Set<() => void>();
function emit() { listeners.forEach((l) => l()); }
function subscribe(cb: () => void) { listeners.add(cb); return () => { listeners.delete(cb); }; }

export function getPosts(): Post[] { return loadPosts(); }
export function getPost(slug: string): Post | undefined { return loadPosts().find((p) => p.slug === slug); }

const commentsBySlugCache = new Map<string, Comment[]>();
export function getComments(slug: string): Comment[] {
  const hit = commentsBySlugCache.get(slug);
  if (hit) return hit;
  const list = loadComments().filter((c) => c.postSlug === slug);
  commentsBySlugCache.set(slug, list);
  return list;
}

export function addComment(input: { postSlug: string; parentId: string | null; author: string; body: string; }) {
  const all = [...loadComments()];
  const c: Comment = { id: `c_${Date.now()}_${Math.random().toString(36).slice(2, 7)}`, createdAt: new Date().toISOString(), ...input };
  all.push(c);
  write(COMMENTS_KEY, all);
  cachedComments = all;
  commentsBySlugCache.clear();
  emit();
  return c;
}

export function toggleFollow() {
  const followed = isBrowser() ? window.localStorage.getItem(FOLLOWED_KEY) === "1" : false;
  const current = read<number>(FOLLOWERS_KEY, 1247);
  const next = followed ? Math.max(0, current - 1) : current + 1;
  write(FOLLOWERS_KEY, next);
  cachedFollowers = next;
  if (isBrowser()) {
    if (followed) window.localStorage.removeItem(FOLLOWED_KEY);
    else window.localStorage.setItem(FOLLOWED_KEY, "1");
  }
  emit();
}

export type Archive = Record<string, Record<string, Post[]>>;
export function getArchive(): Archive {
  if (cachedArchive) return cachedArchive;
  const posts = loadPosts();
  const map: Archive = {};
  for (const p of posts) {
// React uygulamasını ekrandaki root elementine bağlıyoruz
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(React.createElement(App));
