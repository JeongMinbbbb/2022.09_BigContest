# 2022.09_제10회 빅콘테스트 데이터 분석 공모전

## 주제 : 앱 사용성 데이터를 통한 대출신청 예측분석
> 제10회 빅콘테스트 공모전 데이터 분석 분야 퓨처스리그
***
  
### **Data**
 = 2022년 3월 ~ 6월 Data  
    
- user_spec.csv (user 신용정보)
    - 신청서 번호, 유저 번호, 신청 정보, 유저 정보 등 17개의 Column
    - 분석에 활용할 데이터  
    
- loan_result.csv (사용자가 신청한 대출별 금융사별 승인결과)
    - 신청서 번호, 금융사 번호, 상품 번호, 한도, 금리, 신청 여부 **(target)** 등 7개의 Column
    - 예측해야하는 target 변수를 포함한 데이터  
    
- log_data.csv (finda App 로그 정보 )
    - 유저 번호, 행동 명, 행동 일시 등 4개의 Column
    - 유실된 자료가 있다고 주최측에서 공지한 데이터 **(데이터 결함으로 신뢰성의 문제로 분석에 참고용으로만 사용)**  

https://www.bigcontest.or.kr/points/content.php#ct04

![빅콘_데이터구조_크기조정](https://user-images.githubusercontent.com/90736934/209518174-63e72a3e-6779-4419-9370-2db226977caf.png)
***

### **분석 목적**
- 1. 고객 세그멘트별 홈화면 맞춤 메시지 기획
- 2. 특정 기간 내에 대출을 신청할 고객 예측
- 3. 앱 활성화를 위한 인사이트 도출

     
### **분석 과정**
1. 전처리
- 결측인 데이터들은 각각의 성격에 맞게 범주화, 삭제, 예측을 적용함
    1. 나이, 성별 결측은 예측할 수 없고, 삭제하기엔 데이터의 손실이 많아 **범주화**  

    2. 유저 정보에 결측이 있었지만, 해당 유저는 대출을 할 수 없는 청소년 혹은 본 분석 목적에 해당하지 않는 유저이기에 **삭제**   

    3. 신용 점수라는 변수는 연속형 변수라는 특징 and 대출 정보와 관련 높은 변수라고 판단 => 따라서 MissForest 모델을 사용한 **대체** 

2. 고객 특성 분류
- 행동 로그 및 인적 사항 데이터 전처리
- 복잡한 이상치 정의를 위한 실험 진행
- 2단계 군집분석으로 고객의 특성 분류
    - 짧은 기간 내에 조건을 자주 변경하는 고객들을 정보 오입력 그룹으로 정의하고 사전 분류
    - 정보 오입력 그룹 외 고객들을 대상으로 재차 고객 특성 분류
- 고객 특성별 메시지 기획
    - ex) 대출 활성화 고객 : 변경된 대환 대출 금리 정보를 제공하는 사후 관리 서비스를 안내하여 충성 고객으로의 전환을 유도

  0. K-means 방식을 사용
      - k 값을 선택한 군집화 가능
      - 새로운 사용자에 대한 군집 분류시 적절함
      - 대용량 데이터이기 때문에, 계산 시간 단축  
  
  1. 두 단계로 나눈 군집화 / 신뢰성있는 데이터 확보
      - 분석 진행 중, 일관성이 없는 응답 데이터 발견 (ex: 입사년월이 30개인 사람, 2일 동안 6번의 고용형태 변경)
      - 1차 군집화를 통해 이러한 비정상 데이터를 먼저 분류, 이는 범주형 변수의 고유한 값 개수로 판단 ==> **신뢰성을 판단하는 기준 제시**  
  
2. 각 군집 해석에 맞는 적절한 메세지 제안
      - 신규 고객 : 핀다는 대출 관리 서비스를 주로 제공하고 있어요! 다양한 튜토리얼을 이용해 보세요!
      - 잠재적 휴면 고객 : (팝업창 추가 제안)핀다에 ~하는 새로운 서비스가 업데이트 되었어요! 서비스를 이용해보세요!  
      - 대출 관심 고객 : 대출신청 이력이 있으시네요! 바로 대출관리 서비스로 안내해드릴까요?
      - 정보입력 오류고객 : 이전에 입력하신 정보가 자주 변경되었어요! 다시 한 번 확인해주세요.


3. 대출 신청 상품을 예측하는 모델 구축 
- 데이터 불균형 문제 완화를 위해 대출 신청 프로세스의 특징을 반영한 모델링 진행
    - 대출을 실행한 고객의 신청서를 분류한 뒤, 신청서 내 추천된 각 대출 상품에 대한 신청여부를 예측
- 대출 실행 고객의 신청서를 분류할 때, Recall에 더 높은 가중치를 부여한 평가지표 사용
- 최적의 변수들을 선정하여 XGBoost와 LGBM으로 모델 학습

     0. 사용한 모델
         1. 1차 모델링 : Catboost / 모델 학습에 사용한 user_spec 데이터에 범주형 변수가 많았기 때문
         2. 2차 모델링 XGBoost, Catboost, LGBM / 해석의 용이함, 예측 성능을 모두 고려한 머신러닝 모델 사용 / 앙상블을 적용하기 위해 Boosting 기반 모델 3가지를 사용
     
     1. 두 단계로 나눈 모델링 
          ![빅콘_모델구조_크기조정](https://user-images.githubusercontent.com/90736934/209518599-7b2d945f-8f89-4280-949a-77901a465170.png)  
     
         - 1차 모델링을 통해 필요 없는 신청서를 제거했기 때문에, 데이터 불균형을 줄일 수 있었음 **[1:19 ==> 1:9]**
     
     2. Recall을 중요하게 판단
          - 1차 모델링에서 FALSE로 판단한 신청서는 최종 의사결정에서 제외되므로, TRUE를 선정하는 기준을 더 유하게 적용할 필요가 있음.
          - => f1-score가 아닌 f1.5-score를 기준으로 모델의 threshold를 조정  
     
     3. threshold 변경
         - 2차 모델링에서 3가지 모델을 앙상블하는 방법에 threshold를 조정하는 방법을 고안함  

 
### **분석 결과 및 성과**
1. 모델의 최종 성능 F1-Score : 약 0.43
2. 성과 : 320개 팀 중 상위 4% 달성, 데이터 손실률 40% 축소
    
#### 최종 탈락에 대한 피드백(팀원과 리뷰)
    -과적합에 대한 검증과정이 부족했다고 판단됨(모델링 오류)
        -모델링 과정에서 베이지안 최적화 알고리즘을 사용해 하이퍼파라미터 튜닝을 하고 과적합을 방지하려 했음
        -그러나 이 과정에서 K-fold cross validation 처리를 놓쳤고, 발표 당시 모델의 과적합에 대한 피드백을 받음
        -이로 인해 모델링을 2단계로 나눈 것에 대한 장점까지 퇴색됨. 불균형 해소와 추가적인 해석만으로는 방법론 도입의 근거가 부족


***
### Team Project
#### Teaming   (팀장)이승준, 배정민, 이한재, 조용민 ***(in Dongguk_University_Stat)***

***
