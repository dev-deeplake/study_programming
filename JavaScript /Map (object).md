## Map 객체?
- 키-값 쌍을 저장하는데 사용되는 자료구조.
- 동적으로 추가 및 제거되는 키-값 쌍을 관리해야하거나, 모든 키가 문자열이 아닌 경우 주로 사용한다.
- ### 주요 메소드
	- `map.set(key, value)`
		- 맵에 주어진 키와 값을 추가하거나, 키가 이미 맵에 존재하면 그 키의 값을 업데이트
	- `map.delete(key)`
		- 맵에서 지정된 키와 그 키에 연결된 값을 삭제
## 일반 객체와의 차이점
1. 키의 타입
	- 일반 객체 : 문자열이나 심볼만을 키로 사용 가능
	- Map : 어떤 타입의 값이든 키로 사용 가능 (함수, 객체, 기본 타입 등)
2. 순서
	- 일반 객체 : 키에 따른 정렬 순서가 존재하지 않음
	- Map : 삽입된 순서대로 키-값 쌍을 유지함
3. 크기
	- 일반 객체 : '.length' 프로퍼티를 이용할 수 없어 키의 수를 직접 세어야 함
	- Map : '.size' 프로퍼티를 통해 바로 크기를 알 수 있음
4. 상속
	- 일반 객체 : 'Object.prototype'에서 메소드와 프로퍼티를 상속받음 ("toString" 또는 "valueOf" 같은 키를 사용할 때 문제가 될 수 있음)
	- Map : 'Object.prototype'에서 상속받지 않아 어떤 키라도 안전하게 사용할 수 있음
5. 성능
	- 일반 객체 : 크기가 클 때 특정 연산 (주로 검색과 삭제) 에서 비효율적일 수 있음
	- Map : 크기와 관계 없이 일관된 성능을 보임