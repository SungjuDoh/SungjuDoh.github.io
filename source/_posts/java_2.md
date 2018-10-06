---
title: 참조형 매개변수와 잠조형 반환타입
categories:
- 기타
- java
tags:
- java
- 참조형매개변수
- 참조형반환타입
- 참조변수
---
* **기본형 매개변수 (Primitive Type) :** 가본형 값이 복사되는 것, Read only

* **참조형 매개변수 (Reference Type) :** 인스턴스의 주소가 복사되는 것, Read & Write
  ``` java
  class ReferenceParmEx{
    pulic static void main(String[] args){
      int[] x = {10};
      int[][] y = new int[][] {{55, 60, 65}, {85, 90, 95}};
    }
    static void change(int[] x, int[][] y){
      ...
    }
  }
  ```
  예시 코드와 같이 배열을 매개변수로 넘길 때 사용할 수 있다.


* **참조형 반환타입 :** 인스턴스의 주소가 반환되는 것

``` java
import java.util.Scanner;

public class MatraixMult{
	static int n;
	static int[][] a,b;
	public static void main(String[] args) {
		Scanner input = new Scanner(System.in);
		n = input.nextInt();
		a = new int[n][n];
		b = new int[n][n];
		for(int i=0; i<n; i++) {
			for(int j=0; j<n; j++)
				a[i][j] = input.nextInt();
		}
		for(int i=0; i<n; i++) {
			for(int j=0; j<n; j++)
				b[i][j] = input.nextInt();
		}
		int[][] ans = mult(a,b,n);
		for(int i=0; i<n; i++) {
			for(int j=0; j<n; j++)
				System.out.print(ans[i][j]+" ");
			System.out.println();
		}
	}
	static int[][] mult(int[][] a, int[][] b, int n){
		int[][] ans = new int[n][n];
		for(int i=0; i<n; i++) {
			for(int j=0; j<n; j++) {
				for(int k=0; k<n; k++)
					ans[i][j] += a[i][k]*b[k][j];
			}
		}
		return ans;
	}
}
```
  예시코드와 같이 행렬곱의 결과를 구하는 함수에서 다차원 배열을 반환할 수도 있다.
  또한, 배열이나 인스턴스를 활용하면 메서드가 둘 이상의 값을 반환하는 효과가 있다.
