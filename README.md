# catalog
#include <iostream>
#include <fstream>
#include <sstream>
#include <vector>
#include <json/json.h>
#include <map>
#include <cmath>
long long decodeValue(const std::string &value, int base) {
    long long result = 0;
    for (char digit : value) {
        int num = 0;
        if (digit >= '0' && digit <= '9') {
            num = digit - '0';
        } else if (digit >= 'a' && digit <= 'f') {
            num = digit - 'a' + 10;
        }
        result = result * base + num;
    }
    return result;
}

long long lagrangeInterpolation(const std::vector<std::pair<int, long long>> &points) {
    int k = points.size();
    long long constant = 0;

    for (int i = 0; i < k; i++) {
        long long term = points[i].second; 
        long long denom = 1;
        for (int j = 0; j < k; j++) {
            if (i != j) {
                term *= -points[j].first;
                denom *= (points[i].first - points[j].first);
                if (denom == 0) {
                    std::cerr << "Error: Division by zero detected." << std::endl;
                    return -1;
                }
            }
        }
        constant += term / denom;
    }
    return constant;
}

int main() {
    std::ifstream file("input.json");
    Json::Reader reader;
    Json::Value input;

    if (!reader.parse(file, input)) {
        std::cerr << "Error parsing JSON. Ensure file exists and is properly formatted." << std::endl;
        return 1;
    }

    int n = input["keys"]["n"].asInt();
    int k = input["keys"]["k"].asInt();
    std::cout << "Number of roots (n): " << n << ", Minimum required roots (k): " << k << std::endl;

    std::vector<std::pair<int, long long>> points;
    for (int i = 1; i <= n; i++) {
        std::string index = std::to_string(i);
        if (input.isMember(index)) {
            int x = i;
            int base = std::stoi(input[index]["base"].asString(), nullptr, 10);
            std::string encodedValue = input[index]["value"].asString();
            long long y = decodeValue(encodedValue, base);
            points.push_back({x, y});
            std::cout << "Parsed point: (" << x << ", " << y << ") with base " << base << std::endl;
        }
    }

    if (points.size() < k) {
        std::cerr << "Error: Not enough points to perform interpolation." << std::endl;
        return 1;
    }

    std::vector<std::pair<int, long long>> selectedPoints(points.begin(), points.begin() + k);
    long long constantTerm = lagrangeInterpolation(selectedPoints);

    if (constantTerm != -1) {
        std::cout << "Constant term (c): " << constantTerm << std::endl;
    }

    return 0;
}
