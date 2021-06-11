# 모의 담금질 기법(Simulated Annealing)

- 높은 온도에서, 액체 상태인 물질이 온도가 낮아지면서 결정체로 변하는 과정을 모방한 '해 탐색 알고리즘'
- 용융 상태에서는 물질의 분자가 자유로이 움직이는데, 이를 모방하여, 해를 탐색하는 과정도 특정한 패턴없이 이루어짐
- 온도가 점점 낮아지면 분자의 움직임이 점점 줄어들어 결정체가 되는데, 해 탐색 과정도 이와 유사하게 점점 더 규칙적인 방식으로 이루어짐
![20210611_202728](https://user-images.githubusercontent.com/80511265/121703388-ba90d780-cb0d-11eb-97f0-fcd038cd8a06.jpg)
- 이러한 방식으로 해를 탐색하기 위해, 후보해에 이웃하는(이웃해)를 정의해야 함



# 모의 담금질 수행 과정

- 높은 T에서의 초기 탐색은 최솟값을 찾는데도, 확률 개념을 도입하여 이웃해 중에서도 '나쁜 해'로 이동하는 자유로움이 있음
- 낮은 T에서는 점차 탐색은 아래 방향으로 향하여, 위 방향으로 이동하는 확률이 점차 작아짐
- 지역 최적해에서 더이상 아래로 탐색할 수 없는 상황에 이르렀을 때, '운 좋게'위 방향으로 탐색하다가 전역 최적해를 찾음
- 유전자 알고리즘과 마찬가지로 모의 담금질 기법도 항상 전역 최적해를 찾아준다는 보장은 없음
- 모의 담금질 기법 : 하나의 초기 해로부터 탐색이 진행<->유전자 알고리즘 : 여러 개의 후보해를 한 세대로 하여 탐색을 진행
![20210611_202821](https://user-images.githubusercontent.com/80511265/121703491-db592d00-cb0d-11eb-86a7-2d371dc59794.jpg)




# 모의 담금질 알고리즘

> 임의의 후보해 s를 선택한다.
>
> 초기 T를 정한다.
>
> repeat
>
> ​	for i=1 to k(t) { //k(t)는 T에서의 for-루프 반복 횟수이다.
>
> ​	s의 이웃해 중에서 랜덤하게 하나의 해 s'을 선택한다.
>
> ​	d=(s'의 값)-(s의 값)
>
> ​	if(d<0)  //이웃해인 s'가 더 우수한 경우
>
> ​		s<-s'
>
> ​	else //s'가 s보다 우수하지 않은 경우
>
> ​		q<-(0,1) 사이에서 랜덤하게 선택한 수
>
> ​		if(q<p) s<-s' //p는 자유롭게 탐색할 확률
>
> ​	}
>
> ​	T<-aT //1보다 작은 상수 a를 T에 곱하여 새로운 T를 계산
>
> until(종료 조건이 만족될 때까지)
>
> return s

- 모의 담금질 기법은 T가 높을 때부터 점점 낮아지는 것을 확률 p에 반영시켜서 초기에는 탐색이 자유롭다가 점점 규칙적이 되도록 함
- 확률 p는 T에 따라서 변해야 함
- T가 높을 때에는 p도 크게하여 자유롭게 탐색하고, T가 0이 되면 p를 0에 가깝게 만들어서 나쁜 이웃해 s'이 s로 되지 못하도록 해야함
- s-s' 값 d도 p에 반영시켜야 함
- d값이 크면 p를 작게, d값이 작으면 p를 크게 한다. -> d값의 차이가 큼에도 불구하고 p를 크게 하면 그동안 탐색한 결과가 무시되어 랜덤하게 탐색하는 결과를 낳기 때문
- 확률 p는 다음과 같이 정의 ![20210611_210331](https://user-images.githubusercontent.com/80511265/121703624-ffb50980-cb0d-11eb-8363-6d756c01baf8.jpg)
- 모의 담금질 기법으로 해를 탐색하려면, 먼저 후보해에 대한 이웃해를 정의하여야 하는데, 해를 정의하는 방법에는 삽입, 교환, 반전 으로 이렇게 3가지가 있음



# 모의 담금질 기법 알고리즘

## SimulatedAnnealing

```java
package com.company;

import java.util.ArrayList;
import java.util.Random;

public class SimulatedAnnealing {
    private int niter;
    public ArrayList<Double> hist;

    public SimulatedAnnealing(int niter) {
        this.niter = niter;
        hist = new ArrayList<>();
    }

    public double solve(Problem p, double t, double a, double lower, double upper) {
        Random r = new Random();
        double x0 = r.nextDouble() * (upper - lower) + lower;
        return solve(p, t, a, x0, lower, upper);
    }

    public double solve(Problem p, double t, double a, double x0, double lower, double upper) {
        Random r = new Random();
        double f0 = p.fit(x0);
        hist.add(f0);

        for (int i=0; i<niter; i++) {
            int kt = (int) t;
            for(int j=0; j<kt; j++) {
                double x1 = r.nextDouble() * (upper - lower) + lower;
                double f1 = p.fit(x1);

                if(p.isNeighborBetter(f0, f1)) {
                    x0 = x1;
                    f0 = f1;
                    hist.add(f0);
                } else {
                    double d = Math.sqrt(Math.abs(f1 - f0));
                    double p0 = Math.exp(-d/t);
                    if(r.nextDouble() < 0.0001) {
                        x0 = x1;
                        f0 = f1;
                        hist.add(f0);
                    }
                }
            }
            t *= a;
        }
        return x0;
    }
}
```

## Problem

```java
package com.company;

public interface Problem {
    double fit(double x);
    boolean isNeighborBetter(double f0, double f1);
}
```

## 4차 함수 전역 최적점

```java
package com.company;

public class Main {
    public static void main(String[] args) {
        SimulatedAnnealing sa = new SimulatedAnnealing(10);
        Problem p = new Problem() {
            @Override
            public double fit(double x) {
                return x*x*x*x-3x*x*x-2x*x+8x+7;
            
            }

            @Override
            public boolean isNeighborBetter(double f0, double f1) {
                return f0 < f1;
            }
        };
        double x = sa.solve(p, 100, 0.99, 0, -10, 50);
        System.out.println(x);
        System.out.println(p.fit(x));
        System.out.println(sa.hist);
    }
}
```
![0611-1](https://user-images.githubusercontent.com/80511265/121703923-3be86a00-cb0e-11eb-9023-65a98bbb3210.PNG)
![Screenshot_20210611-231320_Samsung Internet](https://user-images.githubusercontent.com/80511265/121703812-26734000-cb0e-11eb-8d92-a6f5cba3f584.jpg)

