# KDT
开个坑

不难很矢

感觉很吃剪剪枝
### P3710 方方方的数据结构
为了不写分块新学一个数据结构不亏
#### code
```cpp
#include <iostream>
#include <algorithm>
#define int long long

using namespace std;

const int MaxN = 150010, mod = 998244353;

struct Q {
	int op, l, r, x;
} q[MaxN];

struct Tag {
	int a, b;
	
	Tag(int ga = 1, int gb = 0) {
		a = ga, b = gb;
	}
	
	friend Tag operator+(const Tag &a, const Tag &b) {
		return Tag(a.a * b.a % mod, (a.b * b.a % mod + b.b) % mod);
	}
};

struct S {
	int x[2], t[2], ch[2], fa;
	Tag sum, tag;
} a[MaxN << 2];

int p[MaxN], lst[MaxN], go[MaxN], n, m, rt, tot;

bool cmp(const int &i, const int &j) {
	return q[i].x < q[j].x;
}

void G(int x, int y) {
	a[x].x[0] = min(a[x].x[0], a[y].x[0]), a[x].x[1] = max(a[x].x[1], a[y].x[1]);
	a[x].t[0] = min(a[x].t[0], a[y].t[0]), a[x].t[1] = max(a[x].t[1], a[y].t[1]);
}

void update(int k) {
	(a[k].ch[0]) && (G(k, a[k].ch[0]), 0);
	(a[k].ch[1]) && (G(k, a[k].ch[1]), 1);
}

void add(int x, Tag w) {
	a[x].sum = a[x].sum + w;
	a[x].tag = a[x].tag + w;
}

void pd(int k) {
	if (a[k].tag.a != 1 || a[k].tag.b) {
		(a[k].ch[0]) && (add(a[k].ch[0], a[k].tag), 0);
		(a[k].ch[1]) && (add(a[k].ch[1], a[k].tag), 1);
		a[k].tag = {1, 0};
	}
}

int build(int l, int r, bool flag = 0) {
	int mid = l + r >> 1;
	(flag) ? nth_element(p + l, p + mid, p + r + 1, cmp) : nth_element(p + l, p + mid, p + r + 1); 
	int x = p[mid];
	go[x] = mid;
	a[mid].x[0] = a[mid].x[1] = q[x].x, a[mid].t[0] = a[mid].t[1] = x;
	if (l != mid) (a[a[mid].ch[0] = build(l, mid - 1, !flag)].fa = mid);
	if (r != mid) (a[a[mid].ch[1] = build(mid + 1, r, !flag)].fa = mid);
	return update(mid), mid;
}

void add(int x, int sa[2], int sb[2], Tag w) {
	if (!x) return;
	if (a[x].x[0] > sa[1] || a[x].x[1] < sa[0]) return;
	if (a[x].t[0] > sb[1] || a[x].t[1] < sb[0]) return;
	if (a[x].x[0] >= sa[0] && a[x].x[1] <= sa[1] && 
	a[x].t[0] >= sb[0] && a[x].t[1] <= sb[1]) return add(x, w);
	if (sa[0] <= q[p[x]].x && q[p[x]].x <= sa[1] && p[x] <= sb[1]) a[x].sum = a[x].sum + w;
	pd(x);
	add(a[x].ch[0], sa, sb, w), add(a[x].ch[1], sa, sb, w); 
}

void DFS(int x) {
	if (!x) return;
	update(x);
	DFS(a[x].fa);
	pd(x);
}

int query(int x) {
	a[x].x[0] = a[x].t[0] = MaxN, a[x].x[1] = a[x].t[1] = 0;
	DFS(x);
	return a[x].sum.b;
}

signed main() {
	cin >> n >> m;
	for (int i = 1; i <= m; i++) {
		cin >> q[i].op;
		if (q[i].op <= 2) {
			cin >> q[i].l >> q[i].r >> q[i].x, lst[i] = MaxN, q[i].x %= mod;
		} else if (q[i].op == 3) {
			cin >> q[i].x;
			p[++tot] = i;
		} else {
			cin >> q[i].x;
			lst[q[i].x] = i;
		}
	}
	rt = build(1, tot);
	for (int i = 1; i <= m; i++) {
		if (q[i].op == 4) continue;
		else if (q[i].op == 3) {
			cout << query(go[i]) << endl;
		} else {
			int sa[2], sb[2];
			sa[0] = q[i].l, sa[1] = q[i].r, sb[0] = i, sb[1] = lst[i];
			add(rt, sa, sb, (q[i].op == 1) ? (Tag(1, q[i].x)) : (Tag(q[i].x, 0)));
		}
	}
	return 0;
}
```

## [P5621 [DBOI2019] 德丽莎世界第一可爱](https://files.cnblogs.com/files/blogs/756166/%E5%BE%B7%E4%B8%BD%E8%8E%8E%E4%B8%96%E7%95%8C%E7%AC%AC%E4%B8%80%E5%8F%AF%E7%88%B1.zip?t=1756441741&download=true)
首先考虑暴力，直接暴力枚举 i, j 然后暴力转移，是一个四维偏序的最长上升子序列，考虑用 kdt 优化，直接求范围内 f 的最大值即可，这个板子应该是最好用的，比较厉害，上面那个比较傻逼

然后这里的查询需要进行一系列剪枝
```cpp
#include <iostream>
#include <algorithm>
#define int long long
 
using namespace std;
 
const int MaxN = 2e5 + 10;
 
struct Node {
	int x[3], l[3], r[3], maxx, v, sz, ls, rs;
} t[MaxN];
 
struct S {
	int a, b, c, d, e;
	
	bool operator<(const S &j) const {
		return a != j.a ? a < j.a : b != j.b ? b < j.b : c != j.c ? c < j.c : d < j.d;
	} 
} a[MaxN];
 
int b[MaxN], f[MaxN], rt[25], tot, top, n, cutdown, ans = -1e18;
 
void update(int k) {
	t[k].maxx = max({t[t[k].ls].maxx, t[t[k].rs].maxx, t[k].v}), t[k].sz = t[t[k].ls].sz + t[t[k].rs].sz + 1;
	for (int i : {0, 1, 2}) {
		t[k].l[i] = t[k].r[i] = t[k].x[i];
		if (t[k].ls) t[k].l[i] = min(t[t[k].ls].l[i], t[k].l[i]), t[k].r[i] = max(t[t[k].ls].r[i], t[k].r[i]);
		if (t[k].rs) t[k].l[i] = min(t[t[k].rs].l[i], t[k].l[i]), t[k].r[i] = max(t[t[k].rs].r[i], t[k].r[i]);
	}
}
 
int build(int l, int r, int c = 0) {
	int mid = l + r >> 1;
	nth_element(b + l, b + mid, b + r + 1, [c](int x, int y) { return t[x].x[c] < t[y].x[c];});
	t[b[mid]].ls = t[b[mid]].rs = 0;
	if (l < mid) t[b[mid]].ls = build(l, mid - 1, (c + 1) % 3);
	if (mid < r) t[b[mid]].rs = build(mid + 1, r, (c + 1) % 3);
	return update(b[mid]), b[mid];
}
 
void DFS(int &x) {
	if (!x) return;
	DFS(t[x].ls), b[++tot] = x, DFS(t[x].rs), x = 0;
}

void upgrade(int sz = 0) {
	for (b[tot = 1] = top; rt[sz]; sz++) {
		DFS(rt[sz]);
	}
	rt[sz] = build(1, tot);
}

void insert(int x[], int w) {
	t[++top].sz = 1, t[top].maxx = t[top].v = w;
	for (int i : {0, 1, 2}) t[top].x[i] = t[top].l[i] = t[top].r[i] = x[i];
	upgrade();
}
 
bool Check(int pl[], int pr[], int l[], int r[]) {
	for (int i : {0, 1, 2}) if (!(l[i] <= pl[i] && pr[i] <= r[i])) return 0;
	return 1;
}
 
int query(int k, int l[], int r[], int res = 0) {
	if (!k || t[k].maxx <= cutdown) return 0;
	for (int i : {0, 1, 2}) if (t[k].l[i] > r[i] || t[k].r[i] < l[i]) return -1;
	if (Check(t[k].l, t[k].r, l, r)) return cutdown = t[k].maxx;
	if (Check(t[k].x, t[k].x, l, r)) cutdown = max(cutdown, res = t[k].v);
	if (t[t[k].ls].maxx > t[t[k].rs].maxx) 
		return max({res, query(t[k].ls, l, r), query(t[k].rs, l, r)});
	return max({res, query(t[k].rs, l, r), query(t[k].ls, l, r)});
}
 
int _19C(int l[], int r[], int res = 0) {
	for (int i = 0; i <= 19; i++) {
		res = max(res, query(rt[i], l, r));
	}
	return res;
}
 
signed main() {
	ios::sync_with_stdio(0), cin.tie(0);
	cin >> n;
	for (int i = 1; i <= n; i++) {
		tot++, cin >> a[tot].a >> a[tot].b >> a[tot].c >> a[tot].d >> a[tot].e;
		if (a[tot].e <= 0) ans = max(ans, a[tot].e), tot--;
		else a[tot].a += MaxN, a[tot].b += MaxN, a[tot].c += MaxN, a[tot].d += MaxN;
	}
	sort(a + 1, a + tot + 1), n = tot;
	for (int i = 1; i <= n; i++) {
		int l[3] = {1, 1, 1}, r[3] = {a[i].b, a[i].c, a[i].d};
		cutdown = 0, f[i] = a[i].e + _19C(l, r), ans = max(ans, f[i]);
		insert(r, f[i]);
	}
	cout << ans << endl;
	return 0;
} 
```

## 简单题

```cpp
#include <iostream>
#include <algorithm>
#define int long long
 
using namespace std;
 
const int MaxN = 1e6 + 10;
 
struct Node {
	int x[3], l[3], r[3], w, v, sz, ls, rs;
} t[MaxN];
 
struct S {
	int a, b, c, d, e;
	
	bool operator<(const S &j) const {
		return a != j.a ? a < j.a : b != j.b ? b < j.b : c != j.c ? c < j.c : d < j.d;
	} 
} a[MaxN];
 
int b[MaxN], f[MaxN], rt[25], tot, top, n, ans = -1e18;
 
void update(int k) {
	t[k].w = t[t[k].ls].w + t[t[k].rs].w + t[k].v, t[k].sz = t[t[k].ls].sz + t[t[k].rs].sz + 1;
	for (int i : {0, 1}) {
		t[k].l[i] = t[k].r[i] = t[k].x[i];
		if (t[k].ls) t[k].l[i] = min(t[t[k].ls].l[i], t[k].l[i]), t[k].r[i] = max(t[t[k].ls].r[i], t[k].r[i]);
		if (t[k].rs) t[k].l[i] = min(t[t[k].rs].l[i], t[k].l[i]), t[k].r[i] = max(t[t[k].rs].r[i], t[k].r[i]);
	}
}
 
int build(int l, int r, int c = 0) {
	int mid = l + r >> 1;
	nth_element(b + l, b + mid, b + r + 1, [c](int x, int y) { return t[x].x[c] < t[y].x[c];});
	t[b[mid]].ls = t[b[mid]].rs = 0;
	if (l < mid) t[b[mid]].ls = build(l, mid - 1, (c + 1) % 2);
	if (mid < r) t[b[mid]].rs = build(mid + 1, r, (c + 1) % 2);
	return update(b[mid]), b[mid];
}
 
void DFS(int &x) {
	if (!x) return;
	DFS(t[x].ls), b[++tot] = x, DFS(t[x].rs), x = 0;
}

void upgrade(int sz = 0) {
	for (b[tot = 1] = top; rt[sz]; sz++) {
		DFS(rt[sz]);
	}
	rt[sz] = build(1, tot);
}

void insert(int x[], int w) {
	t[++top].sz = 1, t[top].v = t[top].w = w;
	for (int i : {0, 1}) t[top].x[i] = t[top].l[i] = t[top].r[i] = x[i];
	upgrade();
}
 
bool Check(int pl[], int pr[], int l[], int r[]) {
	for (int i : {0, 1}) if (!(l[i] <= pl[i] && pr[i] <= r[i])) return 0;
	return 1;
}
 
int query(int k, int l[], int r[], int res = 0) {
	if (!k) return 0;
	for (int i : {0, 1}) if (t[k].l[i] > r[i] || t[k].r[i] < l[i]) return 0;
	if (Check(t[k].l, t[k].r, l, r)) return t[k].w;
	if (Check(t[k].x, t[k].x, l, r)) res = t[k].v;
	return res + query(t[k].ls, l, r) + query(t[k].rs, l, r);
}
 
int _19C(int l[], int r[], int res = 0) {
	for (int i = 0; i <= 19; i++) {
		res += query(rt[i], l, r);
	}
	return res;
} 
 
signed main() {
	ios::sync_with_stdio(0), cin.tie(0);
	cin >> n;
	for (int op, x, y, w, xx, yy, lst = 0; n; n) {
		cin >> op;
		if (op == 3) return 0;
		if (op == 1) {
			cin >> x >> y >> w, x ^= lst, y ^= lst, w ^= lst;
			int l[] = {x, y}; 
			insert(l, w);
		} else {
			cin >> x >> y >> xx >> yy, x ^= lst, y ^= lst, xx ^= lst, yy ^= lst;
			int l[] = {x, y}, r[] = {xx, yy};
			cout << (lst = _19C(l, r)) << '\n';
		}
	}
	return 0;
} 
```
