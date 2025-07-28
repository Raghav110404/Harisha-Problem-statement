# Harisha-Problem-statement#include
<bits/stdc++.h>
#include <boost/multiprecision/cpp_int.hpp>
#include <nlohmann/json.hpp>
using namespace std;
using namespace boost::multiprecision;
using json = nlohmann::json;

// Function to calculate gcd
cpp_int gcdCalc(cpp_int a, cpp_int b) {
    while (b != 0) {
        cpp_int temp = b;
        b = a % b;
        a = temp;
    }
    return a;
}

// Function to calculate lcm
cpp_int lcmCalc(cpp_int a, cpp_int b) {
    return (a / gcdCalc(a, b)) * b;
}

// Function to evaluate the operation based on type
cpp_int evaluateShare(string opType, vector<cpp_int> values) {
    if (opType == "sum") {
        cpp_int ans = 0;
        for (auto v : values) ans += v;
        return ans;
    } else if (opType == "multiply") {
        cpp_int ans = 1;
        for (auto v : values) ans *= v;
        return ans;
    } else if (opType == "hcf") {
        cpp_int ans = values[0];
        for (size_t i = 1; i < values.size(); i++)
            ans = gcdCalc(ans, values[i]);
        return ans;
    } else if (opType == "lcm") {
        cpp_int ans = values[0];
        for (size_t i = 1; i < values.size(); i++)
            ans = lcmCalc(ans, values[i]);
        return ans;
    }
    return 0;
}

// Lagrange Interpolation to reconstruct secret
cpp_int lagrangeInterpolation(vector<pair<cpp_int, cpp_int>> shares, int k) {
    cpp_int secret = 0;
    for (int i = 0; i < k; i++) {
        cpp_int xi = shares[i].first;
        cpp_int yi = shares[i].second;
        cpp_int num = 1, denom = 1;
        for (int j = 0; j < k; j++) {
            if (i == j) continue;
            num *= (0 - shares[j].first);
            denom *= (xi - shares[j].first);
        }
        secret += yi * (num / denom);
    }
    return secret;
}

int main() {
    // Reading JSON input
    ifstream inFile("input.json");
    if (!inFile.is_open()) {
        cerr << "Error: Could not open input.json" << endl;
        return 1;
    }

    json input;
    inFile >> input;

    int n = input["n"];
    int k = input["k"];

    vector<pair<cpp_int, cpp_int>> shares;

    // Parsing all shares
    for (auto &item : input["shares"]) {
        cpp_int key = item["key"].get<int>();
        string opType = item["op"];
        vector<cpp_int> values;
        for (auto &v : item["values"]) values.push_back(v.get<int>());
        cpp_int val = evaluateShare(opType, values);
        shares.push_back({key, val});
    }

    // Generate all combinations of k shares
    vector<cpp_int> secrets;
    vector<int> idx(n);
    fill(idx.begin(), idx.begin() + k, 1);

    do {
        vector<pair<cpp_int, cpp_int>> selected;
        for (int i = 0; i < n; i++)
            if (idx[i]) selected.push_back(shares[i]);

        cpp_int secret = lagrangeInterpolation(selected, k);
        secrets.push_back(secret);

    } while (prev_permutation(idx.begin(), idx.end()));

    // Find the most common secret
    map<string, int> freq;
    string finalSecret;
    int maxFreq = 0;

    for (auto &s : secrets) {
        string str = s.convert_to<string>();
        freq[str]++;
        if (freq[str] > maxFreq) {
            maxFreq = freq[str];
            finalSecret = str;
        }
    }

    cout << "✅ The secret is: " << finalSecret << endl;
    return 0;
}

