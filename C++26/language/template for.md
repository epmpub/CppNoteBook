```c++
#include <tuple>
#include <array>

std::tuple<int, char, double> get_tuple();

struct Data {
  int i;
  char c;
  double d;
};

Data get_data();

consteval Data get_constexpr_data() {
  return Data{1, 'a', 1.0};
}

void use(int);
void use(char);
void use(double);

template<auto Value>
void use_constexpr();

int main()
{
  template for (const auto &val : get_data()) {
    use(val);
  }

  
  template for (static constexpr auto data = get_constexpr_data();
                constexpr auto val : data) {
    use_constexpr<val>();
  }
}
```

