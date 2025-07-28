#include <bits/stdc++.h>
using namespace std;
using ll = long long;
struct Share {
    ll x, y;
    Share(ll _x, ll _y) : x(_x), y(_y) {}
};

// Function to compute modular inverse (used in Lagrange interpolation)
ll modInverse(ll a, ll mod) {
    ll m0 = mod, t, q;
    ll x0 = 0, x1 = 1;
    if (mod == 1) return 0;
    while (a > 1) {
        q = a / mod;
        t = mod;
        mod = a % mod, a = t;
        t = x0;
        x0 = x1 - q * x0;
        x1 = t;
    }
    return x1 < 0 ? x1 + m0 : x1;
}

ll lagrangeInterpolation(const vector<Share>& shares, ll mod = 1e9+7) {
    ll result = 0;
    int k = shares.size();

    for (int i = 0; i < k; ++i) {
        ll xi = shares[i].x;
        ll yi = shares[i].y;
        ll term = yi;
        for (int j = 0; j < k; ++j) {
            if (i != j) {
                ll xj = shares[j].x;
                term = term * (mod - xj) % mod;
                term = term * modInverse((xi - xj + mod) % mod, mod) % mod;
            }
        }
        result = (result + term) % mod;
    }

    return result;
}

void generateCombinations(const vector<Share>& shares, int k, int start, vector<Share>& temp, vector<vector<Share>>& result) {
    if (temp.size() == k) {
        result.push_back(temp);
        return;
    }
    for (int i = start; i < shares.size(); ++i) {
        temp.push_back(shares[i]);
        generateCombinations(shares, k, i + 1, temp, result);
        temp.pop_back();
    }
}

ll recoverSecret(vector<Share> shares, int k, ll mod = 1e9+7) {
    map<ll, int> freq;
    vector<vector<Share>> combinations;
    vector<Share> temp;
    generateCombinations(shares, k, 0, temp, combinations);

    for (auto& comb : combinations) {
        ll secret = lagrangeInterpolation(comb, mod);
        freq[secret]++;
    }

    ll maxFreq = 0, secret = -1;
    for (auto& p : freq) {
        if (p.second > maxFreq) {
            maxFreq = p.second;
            secret = p.first;
        }
    }
    return secret;
}

int main() {
    int n = 5, k = 3;
    vector<Share> shares = {
        Share(1, 147),
        Share(2, 309),
        Share(3, 537),
        Share(4, 825),
        Share(5, 1185)  // Add or modify according to JSON input
    };

    ll secret = recoverSecret(shares, k);
    cout << "Recovered Secret: " << secret << endl;
    return 0;
}
