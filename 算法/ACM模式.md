```c++
//主函数
int main (){
    int N; //矩阵的阶数 N，由于是正整数，故大于或等于 1。
    while (cin >> N) {
        SerpentineMatrix (N);
        {
            cout << ColumnFirstItem << ' ';
        }
        cout << endl; 
    }
    return 0;
}
```

常用文件头

```c++
#include<algorithm>
#include <vector>
#include<iterator>
#include<map> 
#include<math.h>
#include <string>
using namespace std;
```

```c++
#include<math>
int abs(int i);
double fabs(double x);
long labs(long n);

double pow (double base, double exponent);
float pow (float base, float exponent);


```

