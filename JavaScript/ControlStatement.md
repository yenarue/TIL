제어구문
=====
조건문, 반복문, 점프문

# 조건문
`if`, `switch`

# 반복문
`while`, `do/while`, `for`, `for/in`, `for/of`

# 점프문
`break`, `continue`, `return`, `throw`

## 라벨문
자바스크립트에서는 모든 문장에 라벨을 붙일 수 있다.

```javascript
//라벨이름: 문장
loop: while(true) {
    if (confirm("종료?")) {
        break loop;
    }
}
```