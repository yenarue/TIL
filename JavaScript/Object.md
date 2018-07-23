객체
====
객체 리터럴, 생성자로 생성가능.

# 객체리터럴
## 생성 및 프로퍼티 추가/삭제
```javascript
var card = { suit: "하트", rank: "A" };  // 객체리터럴로 생성

card.suit
card["suit"]

// 프로퍼티 추가
card.value = 14;
console.log(card);   // { suit: "하트", rank: "A", value: 14 }

// 프로퍼티 삭제
delete card.rank;
console.log(card);   // { suit: "하트", value: 14 }
```

## `in` 연산자
프로퍼티가 존재하는지 확인하는 연산자

```javascript
var card = { suit: "하트", rank: "A"};
console.log("suit" in card); // true
console.log("value" in card); // false
```

## 프로퍼티 및 메서드 정의
js에서는 메서드도 프로퍼티임

```javascript
var circle = {
    center: { x:1.0, y:2.0 },
    radius: 2.5,
    area: function() {
        return Math.PI * this.radius * this.radius;
    }
};

// 메서드도 프로퍼티이므로
// 이렇게 메서드도 동적으로 추가 할 수 있음ㅋ
circle.translate = function(x, y) {
    this.center.x = this.center.x + x;
    this.center.y = this.center.y + y;
};
```

# 생성자
동일 클래스의 인스턴스를 여러개 만들 수 있다.

```javascript
function Circle(center, radius) {
    this.center = center;
    this.radius = radius;
    this.area = function() {
         return Math.PI * this.radius * this.radius;
    }
}
```