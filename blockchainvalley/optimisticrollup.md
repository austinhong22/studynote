# Optimistic Rollup 심층 리서치

## Optimistic Rollup의 기술적 구조와 작동 방식

Layer 2 실행과 Layer 1 기록: Optimistic Rollup은 이더리움의 레이어 2(L2) 확장 기술로, 대부분의 트랜잭션 처리를 체인 밖(off-chain)에서 수행하고 요약된 결과만 레이어 1(L1, 이더리움 메인넷)에 기록합니다  



롤업의 운영은 L1에 배포된 스마트 컨트랙트 집합에 의해 관리되며, 이 컨트랙트들은 Rollup 체인의 상태(root)와 트랜잭션 데이터를 보관하고 사용자 자산의 입출금을 관리합니다  



롤업 시퀀서(Sequencer) 또는 **운영자(operator)**는 L2에서 사용자 트랜잭션을 모아 순서대로 처리한 뒤, 여러 트랜잭션을 한 번에 **배치(batch)**로 압축하여 L1의 롤업 컨트랙트에 제출합니다  



이 때 개별 트랜잭션들의 상세 데이터는 이더리움 **콜데이터(calldata)**로 게시되거나 별도 블롭(blob) 형태로 올려져, 추후 누구나 이를 활용해 롤업 상태를 재현할 수 있게 합니다  



이를 통해 롤업은 **계산(load)**은 L2에서 수행하면서도 데이터는 L1에 보존하여, L1의 보안에 의존한 확장성을 달성합니다 (약 10~100배 이상의 처리량 개선 효과)  


“낙관적” 실행과 Fraud Proof: Optimistic Rollup이 “Optimistic(낙관적)”이라 불리는 이유는, 매 배치의 트랜잭션 결과가 유효하다는 가정하에 우선 받아들여지기 때문입니다  


즉 롤업은 각 배치의 유효성 증명을 사전에 L1에 제공하지 않고도 일단 적용하며, 대신 사후에 잘못이 발견되면 바로잡는 사기 증명(Fraud Proof) 메커니즘에 의존합니다  


롤업 시퀀서는 일정 시간마다 L1에 롤업의 **새 상태(root)**를 제출하는데, 이 상태가 올바르지 않다면 누구든지 그 배치에 이의를 제기할 수 있습니다. 이를 위해 롤업에는 **챌린지 기간(challenge period)**이라고 불리는 검증 지연 시간이 설정되어 있어, 보통 약 7일 정도의 기간 동안 롤업 상태 업데이트에 대한 이의제기를 허용합니다

이 기간 내에 어떤 참여자가 “해당 배치에 잘못된 트랜잭션이 포함되었다”는 증거를 제시하면, L1의 롤업 컨트랙트는 해당 배치의 트랜잭션을 L1에서 다시 실행하여 결과 상태를 검증합니다  



Fraud Proof는 문제가 된 배치의 트랜잭션과 관련 상태 증명을 포함하며, 재실행한 최종 상태와 시퀀서가 제출한 상태를 비교함으로써 오류 여부를 판단합니다  



만약 재검증 결과 상태가 일치하지 않으면 해당 배치는 사기로 간주되어 롤업 상태 업데이트가 롤백되고, 잘못된 상태를 올린 시퀀서는 **페널티(예치금 슬래시 등 벌칙)**를 받습니다  



반대로 챌린지 기간 동안 이의가 없으면 해당 롤업 배치는 정식 확정되고 L1에도 최종적으로 받아들여집니다  


이러한 구조 덕분에 **롤업 트랜잭션의 최종성(finality)**은 기본적으로 L1의 보안을 상속하지만, 최종 확정까지 챌린지 기간만큼 지연되는 트레이드오프가 있습니다.  
요약하면 Optimistic Rollup은 “선 적용, 후 검증” 방식으로 동작하며, 평소에는 신속한 처리를 하지만 문제가 발견될 경우 온체인 검증을 통해 바로잡는 안전장치를 갖추고 있습니다  


---

## 주요 Optimistic Rollup 프로젝트 비교: Arbitrum, Optimism, Boba Network

이더리움 생태계에서 대표적인 Optimistic Rollup 구현으로 **Arbitrum**, **Optimism**, **Boba Network** 세 가지를 꼽을 수 있습니다. 세 프로젝트 모두 기본적인 롤업 개념은 공유하지만, 아키텍처, 검증 방식, 성능 지표, 생태계 규모, 개발자 친화성 측면에서 몇 가지 차이가 존재합니다. 아래 표는 각 프로젝트의 주요 특성을 비교한 것입니다:

| 구분 | Arbitrum (Offchain Labs) | Optimism (OP Mainnet) | Boba Network (Enya) |
| --- | ------------------------ | --------------------- | ------------------- |
| **아키텍처** | Multi-round Fraud Proof (상호작용式 다단계 이의제기)<br>Nitro 업그레이드로 WASM 기반 Arbitrum VM 사용 (EVM과 등가 수준 지원). 시퀀서 단일 운영 (Arbitrum One) 및 체인 상태 L1에 기록. | Single-round Fraud Proof (단일 단계 이의제기)<br>이더리움 EVM과 1:1 호환되는 OP Stack 기반. 시퀀서 단일 운영 (OP Mainnet) 및 상태 L1 기록. | Optimism의 OVM 코드를 포크하여 구축 (OP Stack 기반). Fraud Proof 구조는 Optimism과 유사 (단일 단계). 이더리움과 EVM 호환. |
| **성능 및 처리량** | 이론적으로 수천 TPS 이상 처리 가능 (약 2,000~4,000 TPS 추정 [CONFIRMO.NET]). 다단계 증명으로 L1 검증 부하가 적어 비용 효율 높음.<br>2023년 2월 롤업 중 최초로 일일 트랜잭션 수가 이더리움 메인넷을 넘어섰음 [RUBIC.EXCHANGE]. | Arbitrum에 근접한 TPS 성능 (약 2,000 TPS 추정 [CONFIRMO.NET])으로 L2 처리. 단계 간소화로 최종 확정이 약간 더 빠를 수 있으나, 잘못된 경우 L1에서 전체 재실행해 가스비 부담 증가 [MEDIUM.COM].<br>2023년 이후 트랜잭션 급증 추세로 일일 처리량에서 Arbitrum과 경쟁함. | 이론상 Optimism과 유사한 성능이지만, 실제 처리량은 낮음. 2021년 말 메인넷 출시 당시 TVL 기준 6위 (~$1.7억)까지 올랐으나, 이후 사용 감소로 TPS 및 활동 저조. 현재는 이더리움 L2 중 비교적 소규모 처리량 유지. |
| **커뮤니티/생태계** | 가장 큰 L2 생태계: TVL 약 100억 달러 이상, L2 시장의 약 40% 점유. 지원 DApp 수 400+개. DeFi 강세(예: GMX, Uniswap, Aave 등)와 NFT/게임 분야 프로젝트 다수. 2023년 자체 토큰 ARB 출시 및 Arbitrum DAO를 통한 거버넌스 운영. | 두 번째로 큰 L2 생태계: TVL 약 50억 달러 수준. DApp 160+개 이상 지원하며, 특유의 Optimism Collective 커뮤니티와 OP 토큰 거버넌스 활성. 주요 DeFi 프로젝트(Uniswap, Synthetix, Curve 등)와 NFT 플랫폼 활동 중. 2023년 Coinbase의 Base 롤업 등 OP Stack 기반 체인들이 “Superchain” 연합 비전 추진. | 비교적 작은 생태계: 피크 시 TVL 약 $13억에서 현재 수천만 달러대로 감소. 일부 DeFi 프로젝트(예: Envelop)와 NFT/게임 시도가 있으나 주류 프로토콜 채택 저조. Boba 토큰(BOBA)을 활용한 거버넌스가 있으나 커뮤니티 규모 한정적. Avalanche, BNB, Fantom 등 다중체인 전개로 사용 범위 확대 시도. |
| **개발자 친화성** | 이더리움과 높은 호환성 유지 (EVM 등가). 솔리디티 등 모든 이더리움 언어 지원. Nitro 업데이트로 개발자들이 일반 EVM처럼 활용 가능. Stylus(Rust, C++ 등 WASM 스마트컨트랙트) 도입 예정으로 개발 옵션 확대. 다만, 초기에는 핵심 코드베이스가 폐쇄적이었으나 현재는 공개되어 Arbitrum Orbit을 통한 서브체인(L3) 구축 지원. | 완전한 EVM 등가성 달성 (과거 OVM에서 2022년 Bedrock 업그레이드로 표준 EVM과 동일한 실행 환경). 솔리디티, Vyper 등 이더리움 스마트컨트랙트 코드를 수정 없이 배포 가능. OP Stack을 오픈소스로 공개하여 개발자들이 이를 기반으로 독자 L2 구축 가능 (예: Base, Mantle 등). RetroPGF 등으로 개발자에 보상하는 친화 정책도 있음. | 이더리움과의 호환성은 Optimism과 동일 (EVM 지원). Optimism 포크이므로 기존 툴 활용 가능. Hybrid Compute 기능을 통한 오프체인 API 호출 등 특화 기능 제공. 다만, 전체적으로는 생태계 축소로 개발자 지원 리소스 및 툴이 부족한 편. |

*주: 위 수치의 일부는 2023년 말~2025년 초 기준이며, 시장 상황에 따라 변동될 수 있습니다. Arbitrum과 Optimism 두 롤업은 단연 리더로 평가되며, Arbitrum은 약간 더 큰 생태계와 기술적 안정성을, Optimism은 개방형 OP Stack 전략으로 여러 체인을 아우르는 확장을 보여줍니다. Boba Network는 초기 기술을 활용했으나 현재는 활용도와 영향력이 낮은 편입니다. 기술적으로 Arbitrum은 다단계 이의제기를 통해 L1 가스비 최적화와 보안을, Optimism은 단순화된 이의제기를 통해 구현 난이도를 낮추려 했습니다. 두 방식 모두 정상 운영 시 사용자 경험에는 큰 차이가 없으며, Ethereum 메인넷 대비 훨씬 저렴한 수수료와 빠른 속도를 제공합니다.

---

## Optimistic Rollup과 ZK-Rollup의 차이점

이더리움 **롤업(Rollup)**은 크게 Optimistic 방식과 영지식 기반(ZK) 방식으로 나뉩니다. 두 방식의 핵심 차이는 트랜잭션 유효성 검증을 언제/어떻게 보장하는가에 있습니다  


Optimistic Rollup은 모든 트랜잭션이 유효하다고 가정하고 결과만 게시하며 사후적으로 오류를 잡아내지만, ZK-Rollup은 각 배치마다 **유효성 증명(Validity Proof)**을 암호학적으로 생성 및 검증하여 사전에 오류 가능성을 차단합니다  


다음과 같은 구체적인 차이가 있습니다:

- **검증 방식**:  
  - Optimistic Rollup은 부정 거래 발생 시에만 비용을 치르는 **사기 증명(Fraud Proof)**을 사용  
  - ZK-Rollup은 모든 거래에 대해 연산 증명을 생성하는 유효성 증명을 사용  
  - 즉, Optimistic은 **“지연된 검증”**, ZK는 **“즉각 검증”**  
  - Optimistic은 검증 비용이 낮지만 잠재적 오류 검출에 사람이 개입해야 하고, ZK는 복잡한 증명 생성으로 연산 비용이 높음  
  

- **보안 모델**:  
  - ZK-Rollup은 모든 상태 업데이트를 수학적으로 증명 후 L1에 제출하여, 즉시 신뢰 가능한 검증을 보장 (탈중앙 노드의 감시 없이도 안전)  
  - Optimistic Rollup은 **“최소 1명의 정직한 감시자”**가 필요하며, 문제가 발생 시 Fraud Proof 제출로 부정을 잡음  
  

- **최종성(Finality) 지연**:  
  - ZK-Rollup은 유효성 증명이 검증되는 즉시 롤업 결과가 최종 확정되어, 출금 등 후속 처리가 신속함  
  - Optimistic Rollup은 챌린지 기간(통상 1주일)을 기다려야 하므로 L2 → L1 출금에 약 1주일 지연  
  → 특히 빠른 유동성이 필요한 경우 Optimistic은 유동성 공급자(LP)를 통한 우회 브리지가 필요할 수 있음

- **확장성 및 수수료**:  
  - 두 롤업 모두 다수의 트랜잭션을 묶어 L1에 제출해 처리량을 높이고 수수료를 절감  
  - Optimistic Rollup은 증명 생성 빈도가 낮아 평균 L1 부하가 적음  
    
  - ZK-Rollup은 매 배치마다 복잡한 증명을 생성하여 오프체인 연산 비용이 큼  
    
  - 다만, 트랜잭션 수가 증가하면 비용이 거래 간 균등 분산되어 Optimistic과 비슷하거나 낮아질 수 있으며, 향후 기술 발전으로 ZK-Rollup의 효율성이 더욱 개선될 전망  
    

- **데이터 가용성**:  
  - 두 롤업 모두 L1에 모든 데이터를 기록하는 경우 동등한 보안을 제공  
  - 단, 일부 ZK 솔루션은 데이터의 일부를 오프체인에 두는 발리디움(validium) 방식을 택할 수 있음

- **EVM 호환성**:  
  - Optimistic Rollup은 이더리움과 거의 동일한 환경을 제공하여 DApp 포팅이 용이했으며, 2021~2022년 일반 목적 롤업 주류로 자리잡음  
  - ZK-Rollup은 초기에는 결제나 단순 토큰 송금 등 특정 용도에 최적화되어 있었으나, 2023년 들어 StarkNet, zkSync Era, Polygon zkEVM, Scroll 등 EVM 호환 ZK-Rollup들이 출시되며 빠르게 발전  
  - 비탈릭 부테린은 “단기적으로는 Optimistic Rollup이 범용 EVM을 선점하겠지만, 중장기적으로는 ZK-Rollup이 모든 use-case에서 승리할 것”이라고 전망  
    
  - ZK 기술이 성숙하면 Optimistic 방식의 챌린지 지연이나 감시 부담 없이 동일한 기능 및 부가적 프라이버시 제공 등이 가능해질 전망

요약하면,  
- Optimistic Rollup은 구현이 비교적 단순하고 EVM 호환이 쉬워 일찍 실용화되었으나 챌린지 지연 및 보안 감시 의존도가 있음  
- ZK-Rollup은 초기 진입 장벽이 높지만, 궁극적으로는 더 강력한 보안성과 빠른 최종성으로 롤업의 종착점이 될 것으로 예견됩니다  
  

---

## Optimistic Rollup의 보안 모델 및 데이터 가용성 문제

**보안 모델 개요**:  
Optimistic Rollup의 보안은 이더리움 L1의 안전성과 적어도 한 명의 정직한 검증자(감시자) 존재에 기반합니다  


구체적으로, 롤업 시퀀서는 블록 제출 전 L1에 일정 금액의 **예치금(bond)**을 걸고, 부정 상태 제출 시 예치금 몰수(slashing)가 발생하도록 설계됩니다  


또한 누구나 롤업 노드를 가동하여 모든 트랜잭션을 검증할 수 있으며, 시퀀서가 제시한 새 상태에 오류가 있을 경우 챌린지 트랜잭션을 통해 Fraud Proof 절차를 개시할 수 있습니다  


이러한 오픈 참여형 검증 구조 덕분에, 롤업 운영자가 악의적으로 행동하더라도 한 명의 정직한 참여자만 있으면 이를 적발하고 막을 수 있다는 것이 핵심 보안 가정입니다  


이더리움 메인넷은 롤업의 분쟁 해결 법정 역할을 하여, 제출된 Fraud Proof를 검증하고 최종 판결을 내립니다  



이때 부정이 입증되면, 시퀀서의 예치금을 소각하거나 롤업에서 배제하는 등의 경제적 페널티가 적용되어 시퀀서가 정직하게 행동하도록 유인합니다  


반대로, 정당한 배치를 근거 없이 Challenge하는 악성 행위자는 챌린지 과정에서 자기 예치금을 잃어 근거 없는 이의제기를 억제합니다.  
결국 **“경제적 게임이론”**과 **“L1의 최종 판결”**에 의해 롤업의 무결성이 지켜집니다  



챌린지 기간과 데이터 가용성: Optimistic Rollup에서 **챌린지 기간(보통 7일)**은 Fraud Proof 제출을 위한 기술적 시간일 뿐 아니라, L1과 L2 전체의 보안을 위한 완충 기간으로 여겨집니다  



약 1주일이라는 시간은, 만약 L1이 51% 공격이나 검열 당할 경우 커뮤니티가 부정한 롤업 상태 확정을 막기 위한 사회적 대응 시간을 제공하기 위함입니다  


예를 들어, 롤업 운영자가 부정한 상태를 올리고 L1 채굴자들과 결탁해 7일 동안 모든 챌린지 트랜잭션을 검열한다면, 이론적으로 잘못된 롤업 상태가 확정될 수 있습니다. 이 기간 동안 커뮤니티는 공격을 인지하고 하드포크 등 대응책을 논의할 시간을 확보하게 됩니다  


최근에는 7일이 너무 길어 UX에 영향을 준다는 논의로 챌린지 기간 단축에 대한 연구도 진행 중입니다  



또한, **데이터 가용성(Data Availability)**은 매우 중요한 요소입니다. 롤업의 상태 재현에 필요한 모든 데이터가 L1에 공개되어야 안전한 검증이 가능합니다  


만약 롤업 운영자가 트랜잭션 데이터를 숨기고 상태 루트만 올린다면, 검증자들은 오류를 파악할 수 없어 Fraud Proof를 생성할 수 없게 됩니다  


이를 방지하기 위해, 정통 롤업(예: Arbitrum, Optimism)은 모든 트랜잭션 정보를 L1에 기록하여 언제든지 롤업 상태를 검증할 수 있도록 설계되어 있습니다  


이런 데이터 투명성 덕분에, 롤업 운영자가 검열이나 중단을 하더라도 다른 노드가 공개 데이터를 바탕으로 블록 생성을 재개할 수 있습니다  


또한 사용자는 L1에 보관된 트랜잭션 데이터를 이용해 자신의 자산 보유 증명(Merkle Proof)을 만들어, 운영자 응답 없이 롤업 컨트랙트를 통해 자산을 직접 인출할 수 있습니다  



심지어, 롤업 운영자가 특정 사용자의 트랜잭션을 계속 무시할 경우, 사용자는 해당 트랜잭션을 L1 롤업 컨트랙트에 직접 제출하여 시퀀서를 우회할 수 있으며, 프로토콜은 이를 유효한 것으로 인정합니다  


이처럼 L1에 데이터를 공개하는 의무는 롤업의 검열 저항성을 크게 높여, Plasma와 달리 안전한 범용 플랫폼으로 자리잡게 한 중요한 요인입니다  


MEV와 중앙화 이슈:  
Optimistic Rollup의 현재 구현은 대체로 중앙화된 시퀀서에 의해 운영되며, 이는 MEV(Maximal Extractable Value, 채굴자/시퀀서 가치 추출) 및 검열 이슈를 내포합니다.  
단일 시퀀서가 모든 거래 순서를 결정하므로, 특정 거래를 임의로 지연시키거나 순서를 변경해 이익을 취할 가능성이 있습니다. 예를 들어, 디파이 거래가 많은 상황에서 시퀀서가 일부 사용자 거래 앞에 자신의 거래를 넣어 프런트러닝을 할 수 있습니다.  
Arbitrum은 공개된 메인넷 **멤풀(mempool)** 없이 선입선처리(FCFS) 방식으로 이를 최소화하고 있습니다  


Optimism은 장기적으로 MEV 옥션(MEVA) 메커니즘을 도입하여 시퀀서 권한을 경매에 부치고, 제한된 범위 내에서만 트랜잭션 순서를 최적화하도록 설계하고 있습니다  


(Optimism Bedrock 업그레이드 이후 프라이빗 멤풀과 MEVA 개념을 도입했으나, 완전한 구현은 진행 중입니다.)  
또한, 두 롤업 모두 다중 시퀀서 또는 시퀀서 민주화 방안을 모색 중이며, Arbitrum은 체인링크의 **공정 순서 서비스(FSS)**를 활용하는 방안을 실험하기도 했습니다  



요컨대, 현 시점에서는 시퀀서가 일부 MEV를 독점할 수 있는 구조이나, 점진적 탈중앙화를 통해 롤업의 트랜잭션 ordering을 더욱 공정하고 투명하게 만들 계획입니다.  
잠재적 공격 벡터:  
- 시퀀서의 부정행위 및 검열  
- 검증자 부재  
- 스마트 컨트랙트 버그 등  
시퀀서가 악의적으로 행동해 잘못된 상태를 올리더라도, 앞서 언급한 메커니즘에 의해 적발 및 처벌이 가능하며 사용자는 자금을 안전하게 회수할 수 있습니다.  
다만, 아무도 챌린지하지 않는다면 (예: 네트워크 전체에 치명적 버그가 있거나 모든 감시자가 공격당해 오프라인 되는 경우) 잘못된 상태가 통과할 위험이 있습니다.  
이를 완화하기 위해 여러 독립 감시 노드(혹은 서비스)와 롤업 팀들의 감사(audit) 및 버그바운티 프로그램이 운영 중이며, 일부 롤업 구현에서는 업그레이드 권한 등 중앙화 요소를 커뮤니티 DAO에 이양하는 등 탈중앙화 로드맵이 진행되고 있습니다  



예컨대, Arbitrum One은 2023년 출시된 ARB 토큰을 통해 Arbitrum DAO가 주요 업그레이드 및 파라미터 결정을 하게 되었고, Optimism 역시 거버넌스를 통해 점진적으로 권한을 분산시키고 있습니다.  
스마트 컨트랙트 보안 측면에서는, 롤업 컨트랙트와 Fraud Proof 로직에 대해 여러 차례 감사 및 포멀 베리피케이션이 진행되었으나, 취약점 발견 시 L1에서의 패치가 필요할 수 있습니다.  
2022년 Optimism에서는 실제로 하나의 버그가 발견되어 긴급 패치된 사례가 있었습니다.  
정리하면, Optimistic Rollup의 보안 모델은 **“Ethereum이 믿을 수 있는 법원이고, 충분한 사람들이 배심원으로 참여한다”**는 비유로 설명할 수 있습니다.  
이더리움이 최종 판결을 내리고, 경제적 인센티브로 누구나 검사자가 되어 부정을 잡아내므로, 롤업은 L1 만큼 안전하게 동작합니다  


다만, 완전한 탈중앙화와 지속적인 검증자 참여, 그리고 철저한 데이터 가용성 확보가 중요합니다  


---

## 실제 사용 사례 및 생태계 발전 상황

Optimistic Rollup들은 이더리움의 핵심 확장 솔루션으로 자리잡으며 다양한 **디앱(DApp)**과 사용자들에게 채택되고 있습니다.  
Arbitrum One과 Optimism Mainnet을 중심으로 2022년 이후 탈중앙 금융(DeFi), NFT 마켓, 블록체인 게임 등 여러 분야에서 활발한 생태계가 형성되었습니다.  
예를 들어, DeFi 분야에서는 이더리움 메인넷의 주요 프로토콜들이 롤업으로 확장되었는데, Arbitrum에는 Uniswap, SushiSwap, Curve, Aave 등 대형 DEX/대출 프로토콜이 배포되었으며, 특히 GMX와 같은 파생상품 플랫폼은 Arbitrum 전용으로 성장하여 2023년에 폭발적인 거래량을 기록했습니다  


Optimism에서는 Synthetix(파생자산 프로토콜), Lyra(옵션 거래), Velodrome(AMM DEX) 등 다양한 디파이 프로젝트가 활발하게 운영되며, Uniswap 등도 동시에 배포되어 유동성 채굴 인센티브를 제공하는 등 사용자들이 몰려들고 있습니다  


두 롤업 모두 이더리움 대비 90% 이상 낮은 수수료와 빠른 승인 속도를 제공하기 때문에, 소액 거래나 고빈도 거래가 필요한 DeFi 사용자들이 대거 유입되고 있습니다  


NFT 및 게임 부문에서도 Optimistic Rollup 활용이 증가하고 있습니다.  
Arbitrum은 Treasure DAO를 중심으로 한 게임/NFT 허브를 구축하여 여러 블록체인 게임과 메타버스 아이템 마켓플레이스가 활동 중이며,  
Optimism 역시 Quix 등 NFT 마켓과 일부 게임 프로젝트에서 롤업 환경을 활용하고 있습니다.  
또한, 소셜 플랫폼 Reddit은 커뮤니티 포인트 시스템을 위해 Arbitrum 기술을 응용한 Arbitrum Nova 체인을 사용한 사례도 있습니다.  
(Arbitrum Nova는 Optimistic Rollup과 유사한 AnyTrust 체인으로, 대규모 소셜 앱에서도 롤업 기술의 활용 가능성을 보여줍니다.)

생태계 지표의 성장:  
다양한 사용 사례 덕분에 Optimistic Rollup의 온체인 지표는 최근 몇 년간 급격히 성장했습니다.  
- 2022년 하반기에는 이더리움 메인넷과 L2 전체 거래량 중 50% 이상이 L2에서 발생하기 시작  
- 2023년 초, Arbitrum의 일일 트랜잭션 처리량이 이더리움 L1을 추월하는 역사적 순간이 있었음  
  

실제로 2023년 2월 21일 Arbitrum One은 약 110만 건 이상의 하루 거래를 기록하며 이더리움 메인넷의 거래량을 넘어섰고,  
Optimism도 이후 일시적으로 Arbitrum을 추월할 정도로 거래량이 증가하는 등 L2 간 경쟁이 치열합니다  


누적 처리 건수 측면에서는, 2023년 기준 Arbitrum이 5억 건 이상의 트랜잭션을, Optimism도 2억 건 이상을 처리하여 수년 만에 수억 건 규모의 사용을 달성했습니다.  
또한, 사용자 수 역시 크게 늘어나, 예를 들어 2023년 3월 Arbitrum의 ARB 토큰 에어드롭에 62.5만 명의 사용자가 참여하였고,  
Optimism의 OP 에어드롭에도 수십만 명이 신청하는 등 수백만 개의 L2 지갑 주소가 존재합니다.  
총 락업 자산(TVL) 기준으로는 2025년 초 현재 이더리움 L2 전체에 약 420억 달러 상당의 자산이 예치되어 있으며,  
그 중 85% 이상이 Optimistic Rollup 계열인 Arbitrum과 Optimism에 분포되어 있습니다  


이는 Optimistic Rollup이 현 시점 L2 생태계의 주류임을 보여주며, 사용자와 개발자 모두에게 활발히 활용되고 있음을 나타냅니다.

미래 전망:  
Optimistic Rollup 생태계는 지속적인 확장 국면에 있습니다.  
- Arbitrum과 Optimism은 각각 Arbitrum Orbit 및 Optimism Superchain 이니셔티브를 통해 다수의 새로운 롤업 체인을 지원할 계획을 발표  
- 2023년, Coinbase 거래소가 Optimism 기술을 활용한 Base 체인을 출시하여 TVL 약 $8억 및 하루 수백만 트랜잭션을 기록하는 등 큰 성공을 거둠  
- OP Stack 계열의 체인 증가로 Optimistic Rollup 기술 표준이 확산되고, 상호 연결되는 멀티 롤업 생태계가 형성될 전망  
- Optimistic Rollup 팀들은 ZK 기술 도입 등 기술 업그레이드에도 힘쓰고 있으며, 예를 들어 Metis는 Optimistic에 ZK 요소를 결합한 Hybrid Rollup을 연구 중임  
- 향후 몇 년간 Optimistic Rollup 기반 L2들은 더 짧은 출금 지연, 높은 탈중앙화, 낮은 수수료(EIP-4844 등 데이터 블롭 도입) 등을 통해 사용자 경험을 개선할 것으로 보임.  
  실제로 2023년 말 이더리움에 도입된 데이터 블롭(Proto-danksharding) 기능으로 롤업 데이터 수수료가 크게 저감되어 L2 이용 비용이 더욱 낮아질 전망입니다.

결론적으로,  
Optimistic Rollup은 다수의 실사용 사례와 견고한 생태계를 바탕으로 기술적으로 계속 진화하며, 이더리움 확장의 주축 역할을 공고히 해 나가고 있습니다.

---

**참고 자료**:  
이 보고서의 내용은 이더리움 재단 문서, 각 프로젝트의 기술 문서, L2beat 등의 최신 통계 자료 및 관련 연구글을 기반으로 정리되었습니다.  
보다 자세한 기술 사항은 Ethereum.org의 롤업 설명서와 각 롤업의 공식 문서(예: docs.arbitrum.io 등)를 통해 확인할 수 있습니다.  
또한, Optimistic Rollup과 ZK-Rollup의 비교에 대해서는 Vitalik Buterin의 롤업 가이드, StarkWare 등 업계 자료 및 L2beat 리서치 등을 참조하였습니다.
