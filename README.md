# Oh, C++
Why must you be like that.

```
#include <numeric>
#include <iostream>
#include <vector>

template <typename... Args>
auto sum(const Args... args) -> decltype((args + ... + 0)) {
  const std::vector<decltype((args + ... + 0))> values = {args...};
  return std::accumulate(values.begin(), values.end(), decltype((args + ... + 0)){0});
}

int main() {
  std::cout << sum(1, 2, 3.5) << std::endl;
}

```
