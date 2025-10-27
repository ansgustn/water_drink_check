# water_drink_check

### 프로젝트 소개
**water drink check**는 사용자가 하루 동안 섭취한 물의 양을 기록하고, 목표 섭취량 달성을 돕는 건강 관리 앱입니다.  
간단한 인터페이스와 시각적인 진행률 표시를 통해 건강한 수분 섭취 습관을 형성하도록 지원합니다.

---

### 주요 기능
1. **목표 물 섭취량 설정** — 사용자 맞춤 목표(예: 2000ml) 설정 가능  
2. **물 추가 기능** — 버튼 클릭으로 마신 물의 양을 간편하게 기록  
3. **진행률 시각화** — 원형 ProgressView를 통해 목표 달성률 표시  
4. **시간별 기록 관리** — 섭취 시각별 기록 자동 저장  
5. **일일 초기화 기능** — 하루가 지나면 자동 초기화, 또는 수동 초기화 가능

---

### 기술 스택
| 구분 | 내용 |
|------|------|
| **언어** | Swift 5.0 |
| **UI 프레임워크** | SwiftUI |
| **저장 방식** | UserDefaults (로컬 데이터 유지) |
| **IDE** | Xcode 15 이상 |
| **지원 OS** | iOS 17 이상 |

---


### 개발 환경
웹 코딩
Online Swift Code Editor and Compiler 환경에서 작성


### 1. Model
import Foundation

// MARK: - Model

/// 물 섭취 기록 개별 항목
struct WaterEntry: Identifiable, Codable {
    let id = UUID()
    let amount: Int // 섭취량 (ml)
    let date: Date // 섭취 시간
    
    var formattedTime: String {
        let formatter = DateFormatter()
        // '오전 7:50' 또는 '오후 4:50' 형식
        formatter.dateFormat = "a h:mm" 
        formatter.locale = Locale(identifier: "ko_KR") // 한국어 로케일 설정
        return formatter.string(from: date)
    }
    
    var formattedDate: String {
        let formatter = DateFormatter()
        // '2025-10-27' 형식
        formatter.dateFormat = "YYYY-MM-dd"
        return formatter.string(from: date)
    }
}

/// 앱 데이터 (목표량, 오늘 섭취 기록)
struct WaterData: Codable {
    var dailyGoal: Int = 2000 // 일일 목표량 (ml)
    var entries: [WaterEntry] = [] // 모든 섭취 기록
    
    var todayEntries: [WaterEntry] {
        // 오늘 날짜 필터링 및 시간순 (최신순) 정렬
        let today = Calendar.current.startOfDay(for: Date())
        return entries
            .filter { Calendar.current.isDateInToday($0.date) }
            .sorted(by: { $0.date > $1.date }) // 최신 기록이 맨 위에 오도록 정렬
    }
    
    var todayTotal: Int {
        todayEntries.reduce(0) { $0 + $1.amount }
    }
}

### 2. ViewModel

import Foundation
import Combine
import SwiftUI

// MARK: - ViewModel

/// 물 섭취 추적 로직 및 데이터를 View에 제공
final class WaterTrackerViewModel: ObservableObject {
    @Published private(set) var data: WaterData // 읽기 전용으로 외부 노출
    
    init(data: WaterData = WaterData()) {
        self.data = data
    }
    
    var dailyGoal: Int {
        data.dailyGoal
    }
    
    var todayTotal: Int {
        data.todayTotal
    }
    
    var completionPercentage: Double {
        Double(todayTotal) / Double(dailyGoal)
    }
    
    var entries: [WaterEntry] {
        data.todayEntries
    }

    /// 물 섭취 기록 추가
    func addWater(amount: Int) {
        let newEntry = WaterEntry(amount: amount, date: Date())
        data.entries.append(newEntry)
    }
    
    /// 기록 초기화 (예: 오늘 기록만)
    func resetTodayEntries() {
        let today = Calendar.current.startOfDay(for: Date())
        // 오늘 기록이 아닌 것만 남김
        data.entries.removeAll { Calendar.current.isDateInToday($0.date) }
    }
    
    // 이 외에 데이터 영구 저장/불러오기 로직 (UserDefaults, CoreData 등) 추가 필요
}

### 3. View

## 3.1 waterEnteryRow

// MARK: - View Components

struct WaterEntryRow: View {
    let entry: WaterEntry
    
    var body: some View {
        HStack {
            VStack(alignment: .leading) {
                Text(entry.formattedDate)
                    .font(.subheadline)
                    .foregroundColor(.gray)
                Text("\(entry.amount)ml")
                    .font(.title3)
                    .fontWeight(.medium)
            }
            Spacer()
            Text(entry.formattedTime)
                .font(.title3)
                .fontWeight(.medium)
        }
        .padding()
        // 이미지에서 보이는 회색 배경과 둥근 모서리 스타일
        .background(Color(.systemGray6)) 
        .cornerRadius(10)
        // 리스트 셀의 기본 인셋을 제거하여 배경이 꽉 차도록 함 (iOS 16+ 에서 필요)
        .listRowInsets(EdgeInsets()) 
        .listRowSeparator(.hidden) // 구분선 숨기기
    }
}

## 3.2 MainView

struct WaterTrackerView: View {
    // 뷰모델 인스턴스: ObservableObject를 구독
    @StateObject var viewModel = WaterTrackerViewModel() 

    // UI에서 사용된 색상과 유사한 색상 정의
    let lightBlue = Color(red: 0.7, green: 0.9, blue: 1.0) 
    let darkBlue = Color(red: 0.3, green: 0.7, blue: 1.0)
    
    var body: some View {
        NavigationView {
            ZStack {
                // 전체 배경색
                LinearGradient(gradient: Gradient(colors: [lightBlue, Color.white]), startPoint: .top, endPoint: .bottom)
                    .ignoresSafeArea()
                
                VStack(spacing: 20) {
                    Text("2025-10-27 오후 11:20") // 현재 시간 표시 (샘플)
                        .font(.subheadline)
                        .padding(.top, 10)
                    
                    // MARK: - 원형 진행률 표시
                    CircularProgressView(
                        progress: viewModel.completionPercentage,
                        totalAmount: viewModel.todayTotal,
                        goalAmount: viewModel.dailyGoal
                    )
                    .frame(width: 150, height: 150)
                    .padding(.bottom, 30)

                    // MARK: - 진행률 바 및 텍스트
                    VStack(alignment: .leading, spacing: 5) {
                        Text("달성률")
                            .font(.headline)
                        
                        HStack {
                            // 리셋 버튼
                            Button("reset") {
                                viewModel.resetTodayEntries()
                            }
                            .buttonStyle(.borderedProminent)
                            .tint(.red)
                            .cornerRadius(5)
                            
                            Spacer()
                            
                            // 현재/목표량 텍스트
                            Text("\(viewModel.todayTotal)ml / \(viewModel.dailyGoal)ml")
                                .font(.body)
                                .fontWeight(.semibold)
                        }
                        
                        // 진행률 바
                        GeometryReader { geometry in
                            let barWidth = geometry.size.width
                            let progressWidth = min(barWidth, barWidth * CGFloat(viewModel.completionPercentage))
                            
                            ZStack(alignment: .leading) {
                                Capsule()
                                    .fill(Color(.systemGray5))
                                    .frame(height: 10)
                                
                                // 그라데이션 진행률
                                Capsule()
                                    .fill(LinearGradient(gradient: Gradient(colors: [lightBlue, darkBlue]), startPoint: .leading, endPoint: .trailing))
                                    .frame(width: progressWidth, height: 10)
                                    .animation(.easeInOut, value: progressWidth)
                            }
                        }
                        .frame(height: 10)
                    }
                    .padding(.horizontal)

                    // MARK: - 기록 목록
                    List {
                        ForEach(viewModel.entries) { entry in
                            WaterEntryRow(entry: entry)
                        }
                    }
                    // List 기본 스타일 제거 및 List 배경을 투명하게 설정
                    .listStyle(.plain) 
                    .scrollContentBackground(.hidden) 
                    
                    Spacer()
                }
                .navigationTitle("Water Tracker")
                .navigationBarTitleDisplayMode(.inline)
            }
        }
        // 테스트를 위한 초기 데이터 추가
        .onAppear {
            if viewModel.entries.isEmpty {
                // 예시 기록 추가 (테스트용)
                viewModel.addWater(amount: 150)
                viewModel.addWater(amount: 100)
                viewModel.addWater(amount: 200)
                viewModel.addWater(amount: 200)
            }
        }
    }
}

// MARK: - 추가 View: 원형 진행률
struct CircularProgressView: View {
    let progress: Double
    let totalAmount: Int
    let goalAmount: Int
    
    var body: some View {
        ZStack {
            // 배경 원형
            Circle()
                .fill(Color(red: 0.9, green: 0.95, blue: 1.0)) // 연한 하늘색 배경
                .overlay(
                    Circle()
                        .stroke(Color.white.opacity(0.5), lineWidth: 10)
                )
            
            // 진행률 채우기
            Circle()
                .trim(from: 0, to: min(progress, 1.0))
                .stroke(
                    LinearGradient(gradient: Gradient(colors: [Color(red: 0.6, green: 0.9, blue: 1.0), Color(red: 0.3, green: 0.7, blue: 1.0)]), startPoint: .top, endPoint: .bottom),
                    lineWidth: 10
                )
                .rotationEffect(.degrees(-90))
                .shadow(color: Color.black.opacity(0.2), radius: 5, x: 0, y: 3)

            // 가운데 '+' (이미지에서는 클릭으로 기록 추가하는 버튼 역할로 추정)
            Image(systemName: "plus")
                .font(.largeTitle)
                .foregroundColor(.white)
        }
    }
}


### 4. swift 프로젝트 구조
Models/ : 앱의 데이터 구조 정의
ViewModels/ : 데이터 상태 관리 및 로직 처리
Views/ : swiftUI 기반의 화면 구성
