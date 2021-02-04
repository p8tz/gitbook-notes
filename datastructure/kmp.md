```c++
int kmp(string t, string p) {
    vector<int> next = get_next(p);
    int i = 0, j = 0;
    // 强转int, (int)-1 > (unsigned int)1
    while (i < (int)t.size() && j < (int)p.size()) {
        if (j == -1 || t[i] == p[j]) {
            i++;
            j++;
        } else {
            j = next[j];
        }
    }
    return j == p.size() ? i - j : -1;
}

vector<int> get_next(string p) {
    vector<int> next(p.size(), 0);
    next[0] = -1;
    int i = 0, j = -1;
    while (i < (int)p.size() - 1) {
        if (j == -1 || p[i] == p[j]) {
            i++;
            j++;
            next[i] = j;
        } else {
            j = next[j];
        }
    }
    return next;
}
```

