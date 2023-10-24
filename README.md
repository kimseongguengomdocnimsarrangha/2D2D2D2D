//C:\Program Files(x86)\FMOD SoundSystem\FMOD Studio API Windows\api\core\inc
//C:\Program Files (x86)\FMOD SoundSystem\FMOD Studio API Windows\api\core\lib\x64\fmod_vc.lib
//fmod_vc.lib
//kernel32.lib;user32.lib;gdi32.lib   -> windows api 설정
#include <iostream>
#include <fmod.hpp> 
#include <thread>
#include <future>
#include <chrono>
#include <cstdlib>
#include <tuple>
#include <array>
#include <deque>
#include <cmath>
#include <algorithm>
#include <string>
#include <vector>
#include <random>
#include <map>
#include <typeinfo>
#include <conio.h>
#include <windows.h>
#include <limits>
#include <fstream>
#include <ctime>

//한계값 설정
#define MAX 1002
#define INF 999999

//스타일링 단축기 설정
#define RED      "\x1b[31m"
#define GREEN    "\x1b[32m"
#define YELLOW   "\x1b[33m"
#define BLUE     "\x1b[34m"
#define MAGENTA  "\x1b[35m"
#define CYAN     "\x1b[36m"
#define GRAY     "\x1b[90m"
#define RESET    "\x1b[0m"
#define BOLDBLACK     "\x1b[1m\x1b[30m"
#define BOLDRED       "\x1b[1m\x1b[31m"
#define BOLDGREEN     "\x1b[1m\x1b[32m"
#define BOLDYELLOW    "\x1b[1m\x1b[33m"
#define BOLDBLUE      "\x1b[1m\x1b[34m"
#define BOLDMAGENTA   "\x1b[1m\x1b[35m"
#define BOLDCYAN      "\x1b[1m\x1b[36m"
#define BLINK "\x1b[5m"
#define DOWN "\x1b[25m"

using namespace std;

//이동방향 설정
const int dx[] = { 1, -1, 0, 0 }; 
const int dy[] = { 0, 0, 1, -1 };

//모든 A*, BFS에서 통용되는 방문기록
int visited[MAX][MAX]; 

int N, M; //시작맵의 크기(이후 다른 맵의 크기도 N, M에 비례됨)
int x, y, mx, my; //플레이어 위치, 보스 위치,
int mp, M_mp; //플레이어, 보스의 스킬포인트
char **grid; //시작맵
int visitedDun[30]; //던전 방문기록

time_t timer = time(NULL); //랜덤값 창출

const int countHabitat = 1; //필드보스 주거지 수
bool isRun_M = false; //M의 활동 여부
bool isAttackingOfo = false; //player의 공격 상태 여부
const int appearanceInterval_M = 10; //보스의 활동 간격
int tmpNx, tmpNy; //이벤트 발생시 변하는 위치 조정
int modMap = 0; //맵의 모드
int mmMod; //M의 활동목표
int nflag = 0; //상황 창의 고정 여부
int M_AttackFlag = 0, M_AttackDirect; //M의 공격 여부, 공격 방향
int o_DistOfBox, M_DistOfo; //플레이어의 박스까지 거리, 보스의 플레이어까지 거리
int deathFlag = 0; //죽었는지 여부
int atx1, atx2, aty1, aty2; //보스의 공격 좌표 설정
int breakWall = 0, directWall = -1; //다음에 벽을 부수는지 여부, 분쇄 방향
int printRow; //맵 출력시 전역 변수와 상호하기 위함
int LabyrinthQauntity, storeQauntity; //던전 수, 상점 수
long long money = 8; //돈(게임재화)

//플레이어와 보스의 기본 스탯 설정
float p_at = 10, p_de = 5, max_p_hp = 100, p_alpha_at = 0, p_alpha_de = 0, p_alpha_hp = 0;
float playerAttackPoint = p_at + p_alpha_at, playerDefensePoint = p_de + p_alpha_de, playerMaxHealthPoint = max_p_hp + p_alpha_hp, playerHealthPoint = playerMaxHealthPoint;
float m_at = 30000, m_de = 20000, max_m_hp = 3000000, m_alpha_at = 0, m_alpha_de = 0, m_alpha_hp = 0;
float MAttackPoint = m_at + m_alpha_at, MDefensePoint = m_de + m_alpha_de, MMaxHealthPoint = max_m_hp + m_alpha_hp, MHealthPoint = MMaxHealthPoint;

int music_l[1000]; //음악 중복 방지를 위한 리스트
string food_list[] = { "썩은 닭가슴살", "맛있는 닭가슴살", "중국산 찹쌀약과",  "국내산 찹쌀약과", "강황밥" }; //기본 음식 종류
vector<tuple<int, string, int> > circum; //상황 창
vector<pair<int, int> > deleteWall_list; //부술 벽의 좌표

map<const char*, int> music_dict; //현재 음악이 틀어져 있는지 체크
map<int, pair<int, int> > habitatDictionary; //필드보스 주거지 딕셔너리
map<string, int> food_dict; //플레이어가 가지고 있는 음식 딕셔너리
map<string, int> M_foodDictionary; //M이 가지고 있는 음식 딕셔너리
map<pair<int, int>, int>  dictionaryOfDun; //던전 딕셔너리

//음악 종류와 음악 ID 설정
void setSound() {
		music_dict["C:\\Users\\happy\\Music\\997FA1385D28524F1D.mp3"] = 0;
		music_dict["C:\\Users\\happy\\Music\\995573385CFFD24F07.mp3"] = 1;
		music_dict["C:\\Users\\happy\\Music\\MP_Answering Machine Beep.mp3"] = 2;
		music_dict["C:\\Users\\happy\\Music\\MP_Bird In Rain.mp3"] = 3;
		music_dict["C:\\Users\\happy\\Music\\MP_Large Metal Pan 2.mp3"] = 4;
		music_dict["C:\\Users\\happy\\Music\\MP_Pellet Gun Pump.mp3"] = 5;
		music_dict["C:\\Users\\happy\\Music\\MP_Shotgun 12ga Fire.mp3"] = 6;
		music_dict["C:\\Users\\happy\\Music\\MP_Pling.mp3"] = 7;
		music_dict["C:\\Users\\happy\\Music\\MP_Computer Error.mp3"] = 8;
		music_dict["C:\\Users\\happy\\Music\\MP_Electronic Chime.mp3"] = 9;
		music_dict["C:\\Users\\happy\\Music\\MP_Japanese Temple Bell Small.mp3"] = 10;
		music_dict["C:\\Users\\happy\\Music\\MP_Decapitation.mp3"] = 11;
		music_dict["C:\\Users\\happy\\Music\\MP_Decapitation Head Fall Off.mp3"] = 12;
		music_dict["C:\\Users\\happy\\Music\\MP_Coin Drop.mp3"] = 13;

		for (int i = 0; i < 7; i++) music_l[i] = 0;
}

//정렬
bool compare1(const int a, const int b) {
		return a < b;
}
bool compare2(const pair<int, int> &a, const pair<int, int> &b) {
		if (a.first == b.first) return a.second < b.second;
		return a.first < b.first;
}

//게임 내 시간 
struct GameTime {
		int gameTimeMinutes = 0;
		int gameTimeHours = 6;
		int gameTimeDaylight = 0;
		int gameTimeNight = 0;
		vector<pair<int, int> > sightPoint;
};

//플레이어 옵션 창 리스트
struct CheckList {
		vector<string> option;
		vector<string> ally;
		vector<string> enemy;
		vector<string> item;
};

//판매 가능한 상품들
struct GlobalSell {
		vector<tuple<string, string, int> > availableSellFood;
		vector<tuple<string, string, int> > availableSellIngredient;
		vector<tuple<string, string, int> > availableSellWeapon;
		vector<tuple<string, string, int> > availableSellArmor;
};

//필드보스 관련 
struct No_type {
		deque<tuple<int, int, char> > No_1;
		vector<tuple<int, int, char> > no1Place;
		vector<pair<int, int> > No1AttackPoint;
		float No1_at = 100, No1_de = 80, max_No1_hp = 3000, No1_hp = 3000;
};

//일반몬스터 관련
struct Mon_type {
		int mon1AttackPoint = 3, mon1DefensePoint = 2, mon1HealthPoint = 50, mon1Cost = 20;
};

//플레이어 공격 방향 여부
struct AttackDirection {
		int attackFlag = 0;
		int frontA = 0, leftA = 0, rightA = 0, downA = 0;
};

GameTime T;
CheckList nowAvailableCheck;
No_type Field;
Mon_type stanCo;
AttackDirection Da;
GlobalSell standard;

//음악 실행 함수
void playSoundEffect(const char* soundPath, int playtime) {
		if (music_l[music_dict[soundPath]]) return;
		else music_l[music_dict[soundPath]] = 1;

		FMOD::System* system;
		FMOD::Sound* sound;
		FMOD::Channel* channel;

		FMOD::System_Create(&system);
		system->init(32, FMOD_INIT_NORMAL, nullptr);

		system->createSound(soundPath, FMOD_DEFAULT, nullptr, &sound);
		system->playSound(sound, nullptr, false, &channel);

		while (true) {
				if (playtime == -50) {
						if (T.gameTimeDaylight == 0 && soundPath == "C:\\Users\\happy\\Music\\MP_Bird In Rain.mp3") break;
						else if (T.gameTimeNight == 0 && soundPath == "?") break;
						continue;
				}
				if (playtime <= 0) break;
				else playtime--;

				bool isNowPlaying = true;
				channel->isPlaying(&isNowPlaying);

		}
		music_l[music_dict[soundPath]] = 0;

		sound->release();
		system->close();
		system->release();
}

//업적 관련 클래스 
class Achievements {
		bool getAchievements_List[1000];
public:
		void setList() {
				for (int i = 0; i < 1000; i++) getAchievements_List[i] = false;
		}
		// (0,0)에 위치하였을 때
		void achievements1() {

				if (getAchievements_List[0] == true) return;

				thread soundThread(playSoundEffect, "C:\\Users\\happy\\Music\\MP_Electronic Chime.mp3", 30000000);
				soundThread.detach();

				circum.push_back({ 0, "???", INF });
				circum.erase(circum.begin());
				getAchievements_List[0] = !getAchievements_List[0];

				return;


		}
		// 사는 품목과 가지고 있는 돈의 차이가 100000배 이상 날때(가격 > 현재 돈)
		void achievements2() {
				if (getAchievements_List[1]) return;

				thread soundThread(playSoundEffect, "C:\\Users\\happy\\Music\\MP_Electronic Chime.mp3", 30000000);
				soundThread.detach();

				circum.push_back({ 1, "???", INF });
				circum.erase(circum.begin());

				getAchievements_List[1] = !getAchievements_List[0];

				return;
		}
};

//커서 위치 변경 함수
void setCursor(int sx, int sy) {
		COORD cursorPosition;
		cursorPosition.X = sx; // X 좌표 (가로)
		cursorPosition.Y = sy;  // Y 좌표 (세로)

		HANDLE hConsole = GetStdHandle(STD_OUTPUT_HANDLE);
		SetConsoleCursorPosition(hConsole, cursorPosition);
}

//게임 시작 전 기본 설정
class Start {
		int puN;
		int puM;
public:

		void setEnvironment() {
				system("mode con:cols=200 lines=90");
				setSound();

				HWND console = GetConsoleWindow();
				ShowWindow(console, SW_MAXIMIZE);
		}

		void explanation() {
				cout << "(1) w : 위 a : 왼 s : 아래 d : 오른쪽" << endl << endl;
				cout << "(2) 2(숫자 2 맞음) : 위로 2칸 q : 왼쪽 2칸 x : 아래 2칸 e : 오른쪽 2칸 -----------  모두 mp 2칸일 때 이동 가능" << endl << endl;
				cout << "(3) # 는 벽이며 벽으로 이동 불가" << endl << endl;
				cout << "(4) " << GREEN "^" RESET << " 는 밟을 시 mp 1 상승" << BLUE "( 2 q x e 로 지나가는 경우에도 mp 상승 불가 )" RESET << endl << endl;
				cout << "(5) M과 o 사이가 가로막혀 있을땐 " << MAGENTA "보라색 문구가 나타납니다." RESET << endl << endl;
				cout << "(6) 현재 나의 위치 =" << CYAN " o" RESET << endl << endl;
				cout << "격자판 크기 설정(N x M) 30 <= N <= 500  40 <= M <= 500" << endl << endl << endl;
		}

		void input() {

				explanation();

				int first_warn = 0;

				while (true) {
						cin >> N >> M;

						if (cin.fail()) {
								cin.clear();
								cin.ignore();

								cout << "int형으로 입력해주세요." << endl;
								continue;
						}
						if (N < 30) {
								cout << "N은 30 이상이여야 합니다. " << endl;
								continue;
						}
						else if (M < 40) {
								cout << "M은 40 이상이여야 합니다. " << endl;
								continue;
						}

						cout << endl << "무작위 맵 선별" << endl;

						for (int i = 0; i < 5; i++) {
								cout << "." << endl;
								if (i == 4) {
										cout << "시작" << endl << endl;
								}
								this_thread::sleep_for(chrono::milliseconds(140)); //500
						}
						if (N >= 35 && M >= 91) modMap = 3;
						else if (N >= 35) modMap = 1;
						else if (M >= 91) modMap = 2;

						puN = N, puM = M;
						if (N >= 35) puN = 35;
						if (M >= 86) puM = 86;

						return;
				}

		}

		void firstCircumPush() {

				for (int i = 0; i < puN; i++) circum.push_back({ -1, " ", -1 });

				thread soundThread(playSoundEffect, "C:\\Users\\happy\\Music\\MP_Pling.mp3", 10000000);
				soundThread.detach();
				circum.push_back({ 1, "M", -4 });

				nowAvailableCheck.enemy.push_back("M"), nowAvailableCheck.ally.push_back("o");

				circum.erase(circum.begin());
		}

		void setFirstAvailableFood(){
				standard.availableSellFood.push_back({ "감자", "Common", 30 });
				standard.availableSellFood.push_back({ "그냥 대파", "Common", 70 });
				standard.availableSellFood.push_back({ "싱싱한 양파", "Common", 80 });
				standard.availableSellFood.push_back({ "토마토", "Common", 85 });
				standard.availableSellIngredient.push_back({ "페인트 물감", "Common", 800 });
				standard.availableSellIngredient.push_back({ "양극 산화 피막", "Uncommon", 2000 });
				standard.availableSellIngredient.push_back({ "카본 코팅제", "Uncommon", 3000});
				standard.availableSellIngredient.push_back({ "세라믹 코팅제", "Rare", 9500 });
		}

		void setFirstAvailableOptions() {
				nowAvailableCheck.option.push_back("아군 확인");
				nowAvailableCheck.option.push_back("적 확인");
				nowAvailableCheck.option.push_back("인벤토리");
		}

		int getPuN() { return puN; }

		int getPuM() { return puM; }

		~Start() { }

};

//필드보스 주거지 디자인
class No_Habitat {
		char habitatType[40][3][3];
		int randomValueX, randomValueY;
public:

		bool place(int px, int py){
				string s = "@_|IPB";
				int continueFlag = 0;

				vector<pair<int, int> > placeVector;
				for (int i = px; i <= px + 2; i++) {
						for (int j = py; j <= py + 2; j++) {
								placeVector.push_back({ i, j });
						}
				}
				
				for (pair<int, int> g : placeVector) {
						int a = g.first, b = g.second;

						bool notfound = s.find(grid[a][b]) == string::npos;

						if ((x != a || y != b) && (mx != a || my != b) && notfound) {} //pass
						else {
								continueFlag = 1;
								break;
						}
				}
				if (continueFlag) return false;

				return true;
		}

		void type1() {
				char habitatType1[3][3] = {
						{'+', '+', '+'},
						{'(', '@', ')'},
						{'+', '+', '+'}
				};
				for (int i = 0; i < 3; ++i) {
						for (int j = 0; j < 3; ++j) {
								habitatType[0][i][j] = habitatType1[i][j];
						}
				}
		}

		void placeBossHabitat() {
				type1();

				char habitat[3][3];
				for (int k = 0; k < countHabitat; k++) { 
						for (int i = 0; i < 3; ++i) {
								for (int j = 0; j < 3; ++j) {
										habitat[i][j] = habitatType[k][i][j];
								}
						}

						while (true) {
								randomValueX = rand() % (N - 10);
								randomValueY = rand() % (M - 10);
								if (!place(randomValueX, randomValueY)) continue;
								else {
										for (int i = randomValueX; i <= randomValueX + 2; i++) {
												for (int j = randomValueY; j <= randomValueY + 2; j++) {
														grid[i][j] = habitat[i - randomValueX][j - randomValueY];
												}
										}

										habitatDictionary[k] = { randomValueX, randomValueY };
										break;
								}
						}
				}

		}
};

//맵과 요소 설정
class MakeMap {
		int randomValueX, randomValueY;
		int x, y, mx, my;
		int numberOfBox = 0;
public:
		No_Habitat Habitat;

		void assignMapArray() {
				grid = new char* [N];
				for (int i = 0; i < N; i++) grid[i] = new char[M];
		}

		void setMap(char **matrix, size_t MaxSizeRow, size_t MaxSizeCol) {

				srand(40002 + (int) timer);
				int cnt = 0;

				for (int i = 0; i < MaxSizeRow; i++) {
						for (int j = 0; j < MaxSizeCol; j++) {
								int ranval = rand() & 3;

								switch (ranval) {
								case 1:
										matrix[i][j] = '^';
										break;
								case 2:
										cnt++;
										matrix[i][j] = '.';
										break;
								case 3:
										matrix[i][j] = '.';
										break;
								default:
										int ranval2 = rand() % 3;
										if (ranval2 <= 1) matrix[i][j] = '#';
										else matrix[i][j] = '.';
								}
						}
				}
		}

		void setQuan() {
				if ((15 <= N && N < 25) || (20 <= M && M < 45)) LabyrinthQauntity = 1, storeQauntity = 2;
				else if ((25 <= N && N < 45) || (45 <= M && M < 80))	LabyrinthQauntity = 2, storeQauntity = 4;
				else if ((45 <= N && N < 70) || (80 <= M && M < 110)) LabyrinthQauntity = 3, storeQauntity = 7;
				else if ((70 <= N && N < 100) || (110 <= M && M < 130)) LabyrinthQauntity = 5, storeQauntity = 10;
				else if ((100 <= N && N < 150) || (130 <= M && M < 155)) LabyrinthQauntity = 6, storeQauntity = 15;
				else if ((150 <= N && N < 200) || (155 <= M && M < 220)) LabyrinthQauntity = 8, storeQauntity = 20;
				else if ((200 <= N && N < 300) || (220 <= M && M < 330)) LabyrinthQauntity = 10, storeQauntity = 30;
				else if ((300 <= N && N < 500) || (330 <= M && M < 590)) LabyrinthQauntity = 14, storeQauntity = 45;
				else if ((500 <= N && N < 750) || (590 <= M && M < 840)) LabyrinthQauntity = 18, storeQauntity = 65;
				else LabyrinthQauntity = 30, storeQauntity = 90;
		}

		void placeLabyrinth() {

				srand(40002);
				vector<pair<int, int> > dunq;
				int LabyrinthCnt = 0;
				while (LabyrinthCnt < LabyrinthQauntity) {

						int placex = rand() % N, placey = rand() % M;
						int rand_d;
						rand_d = rand() % 3;
						int add_and_d = rand() % 3;
						rand_d += add_and_d;
						int flag = 1;

						if ((abs(x - placex) >= 5 || abs(y - placey) >= 5) && (abs(mx - placex) >= 5 || abs(my - placey) >= 5) && (abs(N - placex) >= 4 && abs(M - placey) >= 4)) {

								for (int i = 0; i < dunq.size(); i++) {
										if (abs(placex - dunq[i].first) < 5 && abs(placey - dunq[i].second) < 5) flag = 0;
								}
								if ((abs(0 - placex) < 4 || abs(0 - placey) < 4)) flag = 0;

								if (!flag) continue;

								if (1 <= placex && placex <= N - 1 && 2 <= placey && placey <= M - 2) {

										if (rand_d <= 3) {
												for (int i = 1; i < 4; i++) grid[placex - 1][placey - 2 + i] = '#';
												for (int i = 0; i < 5; i++) grid[placex][placey - 2 + i] = '#';
												grid[placex + 2][placey - 2] = '#', grid[placex + 2][placey + 2] = '#';
												for (int i = 0; i < 5; i++) grid[placex + 3][placey - 2 + i] = '#';
										}
										else if (rand_d == 4) {

												for (int i = placex - 2; i < placex + 3; i++) { for (int j = placey - 2; j < placey + 3; j++) grid[i][j] = 's'; }
												grid[placex - 2][placey - 2] = 'I', grid[placex - 2][placey + 2] = 'I';
												grid[placex - 1][placey - 1] = 'I', grid[placex - 1][placey + 1] = 'I';
												grid[placex + 1][placey - 1] = 'I', grid[placex + 1][placey + 1] = 'I';
												grid[placex + 2][placey - 2] = 'I', grid[placex + 2][placey + 2] = 'I';
										}
										dunq.push_back({ placex, placey });
										dictionaryOfDun[{placex, placey}] = LabyrinthCnt;
										LabyrinthCnt++;
										grid[placex][placey] = '@';
										cout << endl;

								}
						}
				}
		}

		void setPlayerInMap() {
				while (true) {
						randomValueX = rand() % N;
						randomValueY = rand() % M;
						if (grid[randomValueX][randomValueY] != '@' && grid[randomValueX][randomValueY] != '#') {
								x = randomValueX, y = randomValueY;
								break;
						}
				}
		}

		pair<int, int> setPlayerInDun(const int RowN, const int ColM) {
				int totalPossiblity = 2 * (RowN + ColM);
				int randomValue = rand() % totalPossiblity;
				int cnt = 0;
				for (int i = 0; i < RowN; i++) {
						if (cnt == randomValue) return { i, 0 };
						cnt++;
				}
				for (int j = 0; j < ColM; j++) {
						if (cnt == randomValue) return { RowN - 1, j };
						cnt++;
				}
				for (int i = 0; i < RowN; i++) {
						if (cnt == randomValue) return { i, ColM - 1 };
						cnt++;
				}
				for (int j = 0; j < ColM; j++) {
						if (cnt == randomValue) return {0, j };
						cnt++;
				}
		}

		void setBoss() {
				while (true) {
						randomValueX = rand() % N;
						randomValueY = rand() % M;
						if (abs(x - randomValueX) >= N / 2 || abs(y - randomValueY) >= M / 2 && grid[randomValueX][randomValueY] != '@') {
								mx = randomValueX, my = randomValueY;
								break;
						}
				}
		}

		void placeBox() {
				numberOfBox = 0;
				while (numberOfBox < (N * M) / 1000 + (N + M) / 5) {
						randomValueX = rand() % N;
						randomValueY = rand() % M;
						if ((x != randomValueX || y != randomValueY) && (mx != randomValueX || my != randomValueY) && grid[randomValueX][randomValueY] != '#' && grid[randomValueX][randomValueY] != '@') {
								numberOfBox++;
								grid[randomValueX][randomValueY] = 'B';
						}
				}
		}

		void placeStore() {
				int continueFlag;
				int storeCnt = 0;
				while (storeCnt < storeQauntity) {
						continueFlag = 0;
						randomValueX = rand() % (N - 2);
						randomValueY = rand() % (M - 2);
						int p1 = randomValueX + 1, p2 = randomValueY + 1;
						vector<pair<int, int> > placeVector;
						placeVector.push_back({ randomValueX, randomValueY });
						placeVector.push_back({ p1, randomValueY });
						placeVector.push_back({ randomValueX, p2 });
						placeVector.push_back({ p1, p2 });
						for (pair<int, int> g : placeVector) {
								int a = g.first, b = g.second;
								if ((x != a || y != b) && (mx != a || my != b) && grid[a][b] != '#' && grid[a][b] != '@' && grid[a][b] != 'B') { } //pass
								else {
										continueFlag = 1;
										break;
								}
						}
						if (continueFlag) continue;

						grid[randomValueX][randomValueY] = '_', grid[randomValueX][p2] = '_';
						grid[p1][randomValueY] = 'P', grid[p1][p2] = '|';
						storeCnt++;
				}
		}

		void setBossHabitat() { Habitat.placeBossHabitat(); }

		pair<int, int> getPlayerLocation() { return { x, y }; }

		pair<int, int> getBossLocation() { return { mx, my }; }

		int getNumberOfBox() { return numberOfBox; }

		~MakeMap() { };
};

void getMoney(long long value) {
		money += value;
		thread soundThread(playSoundEffect, "C:\\Users\\happy\\Music\\MP_Coin Drop.mp3", 12000000);
		soundThread.detach();
}

//필드보스 
class __No__ {
		bool No_1Flag = false, No_1Wait = false;
		int No1Attack = 0, No_1Down;
public:

		bool checkWallInATile(char type, int add, int px, int py) {
				if (px + add < 0 || px + add >= N || py + add < 0 || py + add >= M) return true;

				if (type == 'X') {
						if (grid[px + add][py] == '#') return true;
				}
				else if (type == 'Y') {
						if (grid[px][py + add] == '#') return true;
				}

				return false;
		}

		char counterAttack(int px, int py) {
				if (px + 1 == x && py == y)	return 'D';
				if (px - 1 == x && py == y) return 'U';
				if (px == x && py - 1 == y) return 'L';
				if (px == x && py + 1 == y) return 'R';
				if (px + 1 == x && py + 1 == y) return 'R';
				if (px + 1 == x && py - 1 == y) return 'L';
				if (px - 1 == x && py + 1 == y) return 'R';
				if (px - 1 == x && py - 1 == y) return 'L';
				return ' ';
		}

		tuple<int, int, int> No_1Bfs(int px, int py, char pDirect, int maxDist, int No1Mod) {
				deque<tuple<int, int, char, int, int, char> > q;
				string s = "|I#+@";
				int turnFlag = 0;
				char take_oDirect = counterAttack(px, py);

				if (take_oDirect != ' ') {
						No1Attack = 1, No_1Wait = 1;
						return { px, py, take_oDirect };
				}

				for (int i = 0; i < N; i++) {
						for (int j = 0; j < M; j++) {
								visited[i][j] = 0;
						}
				}

				if (!No_1Flag) {
						for (int i = px; i < px + 3; i++) {
								for (int j = py; j < py + 3; j++) q.push_back({ i, j, pDirect, -1, -1, '?' });
						}
				}
				else {
						q.push_back({ px, py, pDirect, -1, -1, '?' });
						visited[px][py] = 1;
				}

				if (Field.No_1.size() > 2) turnFlag = 1;

				while (!q.empty()) {
						int nowX, nowY, moveX, moveY;
						char direct, moveDirect;
						tie(nowX, nowY, direct, moveX, moveY, moveDirect) = q.front();
						q.pop_front();

						if (nowX == x && nowY == y) {
								return { moveX, moveY, moveDirect };
						}
						else if ((moveDirect != '?' && moveX != -1 && moveY != -1) && No1Mod == 1 && No_1Wait == false) {
								if (nowX + 1 < N) {
										if (grid[nowX + 1][nowY] == '#') {
												if(checkWallInATile('X', 1, px, py)) No_1Wait = true;
												return { moveX, moveY, moveDirect };
										}
								}
								if (nowX - 1 >= 0) {
										if (grid[nowX - 1][nowY] == '#') {
												if (checkWallInATile('X', -1, px, py)) No_1Wait = true;
												return { moveX, moveY, moveDirect };
										}
								}
								if (nowY + 1 < M) {
										if (grid[nowX][nowY + 1] == '#') {
												if (checkWallInATile('Y', 1, px, py)) No_1Wait = true;
												return { moveX, moveY, moveDirect };
										}
								}
								if (nowY - 1 >= 0) {
										if (grid[nowX][nowY - 1] == '#') {
												if (checkWallInATile('Y', -1, px, py)) No_1Wait = true;
												return { moveX, moveY, moveDirect };
										}
								}
						}
						else if (No1Mod == 1 && No_1Wait == false) {
								if (nowX + 1 < N) {
										if (grid[nowX + 1][nowY] == '#') {
												if (checkWallInATile('X', 1, px, py)) No_1Wait = true;
												return { nowX, nowY, direct };
										}
								}
								if (nowX - 1 >= 0) {
										if (grid[nowX - 1][nowY] == '#') {
												if (checkWallInATile('X', -1, px, py)) No_1Wait = true;
												return { nowX, nowY, direct };
										}
								}
								if (nowY + 1 < M) {
										if (grid[nowX][nowY + 1] == '#') {
												if (checkWallInATile('Y', 1, px, py)) No_1Wait = true;
												return { nowX, nowY, direct };
										}
								}
								if (nowY - 1 >= 0) {
										if (grid[nowX][nowY - 1] == '#') {
												if (checkWallInATile('Y', -1, px, py)) No_1Wait = true;
												return { nowX, nowY, direct };
										}
								}
						}
						else if (visited[nowX][nowY] >= maxDist) continue;

						for (int i = 0; i < 4; i++) {
								int nextX, nextY;
								int alpha;
								if (i == 0 && direct != 'U') {
										if (direct == 'L') nextX = nowX + 2, nextY = nowY + 1, alpha = -1;
										else if (direct == 'R') nextX = nowX + 2, nextY = nowY - 1, alpha = 1;
										else nextX = nowX + 1, nextY = nowY;

										if (nextX < 0 || nextX >= N || nextY - 1 < 0 || nextY + 1 >= M) continue;
										bool notFound = s.find(grid[nextX][nextY]) == string::npos;
										bool notFound2 = s.find(grid[nextX][nextY - 1]) == string::npos;
										bool notFound3 = s.find(grid[nextX][nextY + 1]) == string::npos;

										if ((direct == 'L' || direct == 'R') && turnFlag) {
												bool notFound4 = s.find(grid[nextX - 1][nextY]) == string::npos;
												bool notFound5 = s.find(grid[nextX - 1][nextY + alpha]) == string::npos;
												if (!notFound4 || !notFound5) continue;
										}

										if ((notFound && notFound2 && notFound3) || (notFound && notFound2) || (notFound && notFound3)) {
												if (!visited[nextX][nextY]) {
														if (moveDirect == '?') q.push_back({ nextX, nextY, 'D', nextX, nextY, 'D' });
														else q.push_back({ nextX, nextY, 'D', moveX, moveY, moveDirect });
														visited[nextX][nextY] = visited[nowX][nowY] + 1;
												}
										}
								}
								else if (i == 1 && direct != 'D') {
										if (direct == 'L' && turnFlag) nextX = nowX - 2, nextY = nowY + 1, alpha = -1;
										else if (direct == 'R' && turnFlag) nextX = nowX - 2, nextY = nowY - 1, alpha = 1;
										else nextX = nowX - 1, nextY = nowY;

										if (nextX < 0 || nextX >= N || nextY - 1 < 0 || nextY + 1 >= M) continue;
										bool notFound = s.find(grid[nextX][nextY]) == string::npos;
										bool notFound2 = s.find(grid[nextX][nextY - 1]) == string::npos;
										bool notFound3 = s.find(grid[nextX][nextY + 1]) == string::npos;

										if ((direct == 'L' || direct == 'R') && turnFlag) {
												bool notFound4 = s.find(grid[nextX + 1][nextY]) == string::npos;
												bool notFound5 = s.find(grid[nextX + 1][nextY + alpha]) == string::npos;
												if (!notFound4 || !notFound5) continue;
										}

										if ((notFound && notFound2 && notFound3) || (notFound && notFound2) || (notFound && notFound3)) {
												
												if (!visited[nextX][nextY]) {
														if (moveDirect == '?') q.push_back({ nextX, nextY, 'U', nextX, nextY, 'U' });
														else q.push_back({ nextX, nextY, 'U', moveX, moveY, moveDirect });
														visited[nextX][nextY] = visited[nowX][nowY] + 1;
												}
										}
								}
								else if (i == 2 && direct != 'L') {
										if (direct == 'U' && turnFlag) nextX = nowX + 1, nextY = nowY + 2, alpha = -1;
										else if (direct == 'D' && turnFlag) nextX = nowX - 1, nextY = nowY + 2, alpha = 1;
										else nextX = nowX, nextY = nowY + 1;

										if (nextX - 1 < 0 || nextX + 1 >= N || nextY < 0 || nextY >= M) continue;
										bool notFound = s.find(grid[nextX][nextY]) == string::npos;
										bool notFound2 = s.find(grid[nextX - 1][nextY]) == string::npos;
										bool notFound3 = s.find(grid[nextX + 1][nextY]) == string::npos;

										if ((direct == 'U' || direct == 'D') && turnFlag) {
												bool notFound4 = s.find(grid[nextX][nextY - 1]) == string::npos;
												bool notFound5 = s.find(grid[nextX + alpha][nextY - 1]) == string::npos;
												if (!notFound4 || !notFound5) continue;
										}

										if ((notFound && notFound2 && notFound3) || (notFound && notFound2) || (notFound && notFound3)) {
												if (!visited[nextX][nextY]) {
														if (moveDirect == '?') q.push_back({ nextX, nextY, 'R', nextX, nextY, 'R' });
														else q.push_back({ nextX, nextY, 'R', moveX, moveY, moveDirect });
														visited[nextX][nextY] = visited[nowX][nowY] + 1;
												}
										}
								}
								else if (i == 3 && direct != 'R') {
										if (direct == 'U' && turnFlag) nextX = nowX + 1, nextY = nowY - 2, alpha = -1;
										else if (direct == 'D' && turnFlag) nextX = nowX - 1, nextY = nowY - 2, alpha = 1;
										else nextX = nowX, nextY = nowY - 1;

										if (nextX - 1 < 0 || nextX + 1 >= N || nextY < 0 || nextY >= M) continue;
										bool notFound = s.find(grid[nextX][nextY]) == string::npos;
										bool notFound2 = s.find(grid[nextX - 1][nextY]) == string::npos;
										bool notFound3 = s.find(grid[nextX + 1][nextY]) == string::npos;

										if ((direct == 'U' || direct == 'D') && turnFlag) {
												bool notFound4 = s.find(grid[nextX][nextY + 1]) == string::npos;
												bool notFound5 = s.find(grid[nextX + alpha][nextY + 1]) == string::npos;
												if (!notFound4 || !notFound5) continue;
										}

										if ((notFound && notFound2 && notFound3) || (notFound && notFound2) || (notFound && notFound3)) {
												if (!visited[nextX][nextY]) {
														if (moveDirect == '?') q.push_back({ nextX, nextY, 'L', nextX, nextY, 'L' });
														else q.push_back({ nextX, nextY, 'L', moveX, moveY, moveDirect });
														visited[nextX][nextY] = visited[nowX][nowY] + 1;
												}
										}
								}
						}
				}

				return { -1, -1, '?' };


		}
		//송충이 이동 경로, 왠만하면 건들지 않기
		void No1Attacking(int hx, int hy, char direct) {
				if (direct == 'U') {
						for (int i = 0; i < 3; i++) {
								for (int j = -3 + i; j < 4 - i; j++) {
										if (hy + j < 0 || hy + j >= M || hx - i < 0) continue;
										Field.No1AttackPoint.push_back({ hx - i, hy + j });
								}
						}
				}
				else if (direct == 'D') {
						for (int i = 0; i < 3; i++) {
								for (int j = -3 + i; j < 4 - i; j++) {
										if (hy + j < 0 || hy + j >= M || hx + i >= N) continue;
										Field.No1AttackPoint.push_back({ hx + i, hy + j });
								}
						}
				}
				else if (direct == 'L') {
						for (int i = 0; i < 3; i++) {
								for (int j = -3 + i; j < 4 - i; j++) {
										if (hx + j < 0 || hx + j >= N || hy - i < 0) continue;
										Field.No1AttackPoint.push_back({ hx + j, hy - i });
								}
						}
				}
				else if (direct == 'R') {
						for (int i = 0; i < 3; i++) {
								for (int j = -3 + i; j < 4 - i; j++) {
										if (hx + j < 0 || hx + j >= N || hy + i >= M) continue;
										Field.No1AttackPoint.push_back({ hx + j, hy + i });
								}
						}
				}
		}

		void No_1Run() {
				if (No_1Down == 1) {
						No_1Down = 0;
						return;
				}

				int fno1x, fno1y, maxDist;
				char fno1direct;
				if(!No_1Flag) fno1x = habitatDictionary[0].first, fno1y = habitatDictionary[0].second, fno1direct = '?', maxDist = 11;
				else tie(fno1x, fno1y, fno1direct) = Field.No_1.front(), maxDist = 80;

				tuple<int, int, int> nextMove = No_1Bfs(fno1x, fno1y, fno1direct, maxDist, No1Attack);

				int moveX, moveY;
				char moveDirect;
				tie(moveX, moveY, moveDirect) = nextMove;

				if ((moveX == -1 && moveY == -1 && moveDirect == '?') || No_1Wait) {
						if (No_1Flag && !No1Attack) No1Attack = 1;
						else if (No1Attack) {
								if (No_1Wait) {
										Field.No_1.push_front(nextMove);
										if (Field.No_1.size() > 25) Field.No_1.pop_back();
										No_1Wait = false;
										No1Attacking(moveX, moveY, moveDirect);
										No1Attack = 0;
										No_1Down = 1;
										}
								}
						else{
								if (!(moveX == -1 && moveY == -1 && moveDirect == '?')) {
										Field.No_1.push_front(nextMove);
										if (Field.No_1.size() > 25) Field.No_1.pop_back();
								}
								
						}
						return;
				}

				No1Attack = 0, No_1Wait = false;
				Field.No1AttackPoint.clear();

				if (!No_1Flag) {
						No_1Flag = true;
						thread soundThread(playSoundEffect, "C:\\Users\\happy\\Music\\MP_Japanese Temple Bell Small.mp3", 60000000);
						soundThread.detach();
						circum.push_back({ 1, "털 없는 송충이", -4 });
						circum.erase(circum.begin());
						nowAvailableCheck.enemy.push_back("털 없는 송충이");
				}
				Field.No_1.push_front(nextMove);
				if (Field.No_1.size() > 25) Field.No_1.pop_back();
		}

		bool checkNo1Existence(int i, int j) {
				for (auto coordination : Field.No_1) {
						int nodeX = get<0>(coordination), nodeY = get<1>(coordination);
						if (i == nodeX && j == nodeY) return true;
				}
				return false;
		}

};

//몬스터 
class Mon{
		string stringAboutCant = "|I#+@";
		int mon1Count = 0, mon1Interval = 20;
		int mon1Flag = false;
public:
		vector<array<int, 6> > vectorOfMon1;

		bool intervalMon1() {
				if (!mon1Interval) {
						srand(x + y + 4200);
						mon1Interval = (rand() % 17) + 11;
						return true;
				}
				else {
						mon1Interval--;
						return false;
				}			
		}

		void createMon1() {
				if (mon1Count > 5) return;
				int mon1X, mon1Y;
				int cnt = 0;
				int cant = 0;
				while (true) {
						cnt++;
						if (cnt > 100) return;
						bool cant = false;
						int plus = rand() % 47, minus = -(rand() % 47);
						int moX = x + plus + minus, moY = y + plus + minus;

						if (moX < 0 || moX >= N || moY < 0 || moY >= M || (moX == x && moY == y) || (moX == mx && moY == my) || stringAboutCant.find(grid[moX][moY]) != string::npos) continue;
						for (auto arr : vectorOfMon1) {
								if (arr[0] == moX && arr[1] == moY) cant = 1;
						}
						if (cant) continue;

						mon1X = moX, mon1Y = moY;
						mon1Count++;
						break;
				}
				vectorOfMon1.push_back({mon1X, mon1Y, stanCo.mon1AttackPoint, stanCo.mon1DefensePoint, stanCo.mon1HealthPoint, stanCo.mon1Cost });

				if (!mon1Flag) {
						mon1Flag = true;
						thread soundThread(playSoundEffect, "C:\\Users\\happy\\Music\\MP_Pling.mp3", 10000000);
						soundThread.detach();
						circum.push_back({ 0, "동굴 벌레", -4 });
						circum.erase(circum.begin());
						nowAvailableCheck.enemy.push_back("동굴 벌레");
				}
		}

		void checkDeathMon1() {
				int num = 0;
				vector<int> deathNoteMob1;
				for (array<int, 6> arr : vectorOfMon1) {
						if (arr[4] <= 0) deathNoteMob1.push_back(num);
						num++;
				}
				num = NULL;

				for (int i = deathNoteMob1.size() - 1; i >= 0; i--) {
						circum.push_back({ 1, "동굴 벌레", -INF });
						circum.push_back({ (long long)vectorOfMon1[deathNoteMob1[i]][5], "동굴 벌레", -INF + 1});
						getMoney((long long)vectorOfMon1[deathNoteMob1[i]][5]);
						for (int l = 0; l < 2; l++) circum.erase(circum.begin());

						vectorOfMon1.erase(vectorOfMon1.begin() + deathNoteMob1[i]);
				}
				deathNoteMob1.clear();

		}
		void mon1CheckAndGo(int &hx, int &hy, int z) {
				if (z == 0 && (hx + 1 != x || hy != y) && grid[hx + 1][hy] != '#') hx++;
				else if (z == 1 && (hx - 1 != x || hy != y) && grid[hx - 1][hy] != '#') hx--;
				else if (z == 2 && (hx != x || hy + 1 != y) && grid[hx][hy + 1] != '#') hy++;
				else if (z == 3 && (hx != x || hy - 1 != y) && grid[hx][hy - 1] != '#') hy--;
		}

		void moveRandom(int& hx, int& hy, vector<int> arrRandom) {
				int checkX = hx, checkY = hy;
				for (int i : arrRandom) {
						if (hx + dx[i] < 0 || hx + dx[i] >= N || hy + dy[i] < 0 || hy + dy[i] >= M) continue;
						mon1CheckAndGo(hx, hy, i);
						if (checkX != hx || checkY != hy) return;
				}

		}

		void mon1Bfs(int &hx, int &hy, vector<int> arrRandom) {
				deque<tuple<int, int, int> > q;

				for (int i = 0; i < N; i++) {
						for (int j = 0; j < M; j++) {
								visited[i][j] = -1;
						}
				}

				q.push_back({ hx, hy, -1});
				visited[hx][hy] = 0;

				while (!q.empty()) {
						int nowX, nowY, firstDirection;
						tie(nowX, nowY, firstDirection) = q.front();
						q.pop_front();

						if (nowX == x && nowY == y) {
								mon1CheckAndGo(hx, hy, firstDirection);
								return;
						}
						if (visited[nowX][nowY] > 5) continue;

						for (int i : arrRandom) {
								int cant = 0;
								int nextX = nowX + dx[i], nextY = nowY + dy[i];
								if (nextX < 0 || nextX >= N || nextY < 0 || nextY >= M || (nextX == mx && nextY == my) || visited[nextX][nextY] != -1) continue;
								if (stringAboutCant.find(grid[nextX][nextY]) != string::npos) continue;
								for (array<int, 6> arr : vectorOfMon1) { if (nextX == arr[0] && nextY == arr[1]) cant = 1; }
								if (cant) continue;

								if(firstDirection == -1) q.push_back({ nextX, nextY, i});
								else q.push_back({ nextX, nextY, firstDirection });

								visited[nextX][nextY] = visited[nowX][nowY] + 1;
						}
				}

				return;
		}

		void activeMon() {
				if (mon1Count == 0) return;
				int num = 0;
				for (array<int, 6> arr : vectorOfMon1) {
						vector<int> arrRandom;
						int add = 1;

						while (true) {
								int cant = 0;
								if (arrRandom.size() == 4) break;

								srand(4002 + x + y + my - mx + add);
								int randV = rand() % 4;
								add++;
								for (int i = 0; i < arrRandom.size(); i++) {
										if (arrRandom[i] == randV) cant = 1;
								}
								if (cant) continue;
								arrRandom.push_back(randV);
						}

						int mon1X = arr[0], mon1Y = arr[1];
						if (abs(x - mon1X) > 6 || abs(y - mon1Y) > 9) moveRandom(mon1X, mon1Y, arrRandom);
						else mon1Bfs(mon1X, mon1Y, arrRandom);

						vectorOfMon1[num][0] = mon1X, vectorOfMon1[num][1] = mon1Y;
						num++;
				}
		}

		int checkMob1Existence(int i, int j) {
				int num = 0;
				for (array<int, 6> arr : vectorOfMon1) {
						if (arr[0] == i && arr[1] == j) return num;
						num++;
				}
				return -1;
		}
};

//플레이어 공격 지점 좌표 설정
pair<int, int> playerAttack() {
		if (Da.frontA == 1) return { x - 2, y };
		else if (Da.downA == 1) return { x + 2, y };
		else if (Da.leftA == 1) return { x, y - 2 };
		else if (Da.rightA == 1) return { x, y + 2 };
}

//출력하는 맵의 범위, 공격 지점 범위 설정
tuple<int, int, int, int> monitor(int nowx, int nowy, int add_value1, int add_value2, int sMaxN, int sMaxM) {
		int nextx1 = nowx - add_value1, nextx2 = nowx + add_value1;
		int nexty1 = nowy - add_value2, nexty2 = nowy + add_value2;

		while (true)
		{
				int flag = 1;
				if (nextx1 < 0 || nextx2 >= sMaxN || nexty1 < 0 || nexty2 >= sMaxM) {
						flag = 0;
						if (nextx1 < 0) nextx1 += 1, nextx2 += 1;
						if (nextx2 >= sMaxN) nextx1 -= 1, nextx2 -= 1;
						if (nexty1 < 0) nexty1 += 1, nexty2 += 1;
						if (nexty2 >= sMaxM) nexty1 -= 1, nexty2 -= 1;
				}
				if (flag) break;
		}
		return { nextx1, nextx2, nexty1, nexty2 };
}
tuple<int, int, int, int> attPoint(int attp, int mx, int my, int sMaxN, int sMaxM) {
		if (attp == 0) return monitor(mx - 2, my, 1, 1, sMaxN, sMaxM);
		else if (attp == 1) return monitor(mx + 2, my, 1, 1, sMaxN, sMaxM);
		else if (attp == 2) return monitor(mx, my - 2, 1, 1, sMaxN, sMaxM);
		else if (attp == 3) return monitor(mx, my + 2, 1, 1, sMaxN, sMaxM);
}

//보스의 이동 여부
void M_CheckAndGo(int z) {
		if (z == 0 && (mx + 1 != x || my != y) && grid[mx + 1][my] != '#') mx++;
		else if (z == 1 && (mx - 1 != x || my != y) && grid[mx - 1][my] != '#') mx--;
		else if (z == 2 && (mx != x || my + 1 != y) && grid[mx][my + 1] != '#') my++;
		else if (z == 3 && (mx != x || my - 1 != y) && grid[mx][my - 1] != '#') my--;
}

//모든 캐릭터의 이동 여부
bool nextCheck(const int MaxN, const int MaxM, const int nx, const int ny, const char next, int mod, int fx, int fy, char**matrix, Mon & Mal) {
		for (int i = 0; i < Field.No_1.size(); i++) { if (nx == get<0> (Field.No_1[i]) && ny == get<1> (Field.No_1[i])) return false; }
		for (int i = 0; i < Mal.vectorOfMon1.size(); i++) { if (nx == get<0>(Mal.vectorOfMon1[i]) && ny == get<1>(Mal.vectorOfMon1[i])) return false; }
		if (nx < 0 || nx >= MaxN || ny < 0 || ny >= MaxM) return false;
		if (M_AttackFlag && nx == mx && ny == my) return false;
		if (matrix[nx][ny] == '#' || matrix[nx][ny] == 'I' || matrix[nx][ny] == '+') {
				if (breakWall && mod == 2) return true;
				else return false;
		}

		if (next == '2') { if (matrix[nx + 1][ny] == '#') return false; }
		if (next == 'q') { if (matrix[nx][ny + 1] == '#') return false; }
		if (next == 'e') { if (matrix[nx][ny - 1] == '#') return false; }
		if (next == 'x') { if (matrix[nx - 1][ny] == '#') return false; }

		return true;

}

//플레이어, 보스 이동 bfs
int moveBfs(int px, int py, int fx, int fy, int mod, char**matrix, int MaxN, int MaxM, Mon &Mal) {
		deque<tuple<int, int, int> > q;

		for (int i = 0; i < N; i++) {
				for (int j = 0; j < M; j++) {
						visited[i][j] = -1;
				}
		}

		visited[px][py] = 0;
		q.push_back({ px, py, -1 });

		while (!q.empty()) {
				int currentX, currentY, z;
				tie(currentX, currentY, z) = q.front();
				q.pop_front();
				if (mod == 1 && grid[currentX][currentY] == 'B') {
						return visited[currentX][currentY];
				}
				else if (mod == 2 && currentX == fx && currentY == fy && !mmMod) {
						M_CheckAndGo(z);
						return visited[currentX][currentY] - 1;
				}
				else if (mod == 2 && grid[currentX][currentY] == 'B' && mmMod) {
						M_CheckAndGo(z);
						return visited[currentX][currentY] - 1;
				}
				else if (mod == 2 && grid[currentX][currentY] == '#' && breakWall && fx == -1 && fy == -1) {
						M_CheckAndGo(z);
						return -1;
				}
				else if (visited[currentX][currentY] >= 155) return -1;

				for (int i = 0; i < 4; i++) {
						int nextX = currentX + dx[i], nextY = currentY + dy[i];
						if (!nextCheck(MaxN, MaxM, nextX, nextY, 'U', mod, fx, fy, matrix, Mal)) continue;
						if ((visited[nextX][nextY] == -1 || visited[currentX][currentY] + 1 < visited[nextX][nextY])) {
								if (mod == 2 && currentX == px && currentY == py) {
										q.push_back({ nextX, nextY, i });
								}
								else q.push_back({ nextX, nextY, z });
								visited[nextX][nextY] = visited[currentX][currentY] + 1;
						}

				}
		}

		return -1;

}

//상자 얻었을 때
void getBox(int nowx, int nowy, int who, int& numberOfBox)  {
		thread soundThread(playSoundEffect, "C:\\Users\\happy\\Music\\MP_Pellet Gun Pump.mp3", 23000000);
		soundThread.detach();

		numberOfBox--;
		grid[nowx][nowy] = '.';
		srand(40001 + nowx - nowy);

		int rand_g = rand() % 3;
		int add_and_g = rand() % 3 + 1;
		rand_g += add_and_g;

		int rand_q = rand() % 3;
		int add_and_q = rand() % 2;
		rand_q += add_and_q;
		if (!rand_q) rand_q = 1;

		circum.push_back({ rand_q, food_list[rand_g - 1], who });
		circum.erase(circum.begin() + 0);

		bool isFind = false;

		for (int i = 0; i < nowAvailableCheck.item.size(); i++) {
				if (nowAvailableCheck.item[i] == food_list[rand_g - 1]) {
						isFind = true;
						break;
				}
		}
		if (!isFind && who == 1) nowAvailableCheck.item.push_back(food_list[rand_g - 1]);

		if (who == 1) food_dict[food_list[rand_g - 1]] += rand_q;
		else if (who == 2) M_foodDictionary[food_list[rand_g - 1]] += rand_q;

		nflag = 1;
}

//보스가 부술 벽 체크
int wallCheck(int ai, int aj, int al) {
		if (directWall == 0) {
				if (ai > mx && ai <= mx + al && aj == my) return 1;
		}
		else if (directWall == 1) {
				if (ai < mx && ai >= mx - al && aj == my) return 1;
		}
		else if (directWall == 2) {
				if (aj > my && aj <= my + al && ai == mx) return 1;
		}
		else if (directWall == 3) {
				if (aj < my && aj >= my - al && ai == mx) return 1;
		}
		return 0;
}

//더블 버퍼링
void clearScreen() {
		HANDLE hOut;
		COORD Position;

		hOut = GetStdHandle(STD_OUTPUT_HANDLE);

		Position.X = 0;
		Position.Y = 0;
		SetConsoleCursorPosition(hOut, Position);
}

//필드보스 주거지 출력
void drawHabitat(const int num, const char tile) {
		if (num == 0) {
				if (tile == '+') cout << BOLDGREEN "+" RESET;
				else if (tile == '@') cout << RED "@" RESET;
				else if (tile == ')') cout << GREEN ")" RESET;
				else if (tile == '(') cout << GREEN "(" RESET;
		}
}

//보스를 제외한 적 출력
class DepictNo1 {

public:
		bool printNo1(int i, int j, int playerAttackX, int playerAttackY) {
				for (int k = 0; k < Field.No_1.size(); k++) {
						int no1X, no1Y;
						char no1Direct;
						tie(no1X, no1Y, no1Direct) = Field.No_1[k];
						if (no1X == i && no1Y == j) {

								if (k != 0 && Field.No_1.size() > 2 && get<2>(Field.No_1[k - 1]) != no1Direct) {
										cout << BOLDGREEN;
										if (i >= playerAttackX - 1 && i <= playerAttackX + 1 && j >= playerAttackY - 1 && j <= playerAttackY + 1) cout << CYAN;
										cout << "o" RESET;
								}
								else if (k == 0) {
										cout << BOLDRED;
										if (i >= playerAttackX - 1 && i <= playerAttackX + 1 && j >= playerAttackY - 1 && j <= playerAttackY + 1) cout << CYAN;
										cout << "O" RESET;
								}
								else if ((2 <= k && k <= 5) || (13 <= k && k <= 16) || (20 <= k && k <= 23)) {
										cout << RED;
										if (i >= playerAttackX - 1 && i <= playerAttackX + 1 && j >= playerAttackY - 1 && j <= playerAttackY + 1) cout << CYAN;
										cout << "o" RESET;
								}
								else {
										cout << RED;
										if (i >= playerAttackX - 1 && i <= playerAttackX + 1 && j >= playerAttackY - 1 && j <= playerAttackY + 1) cout << CYAN;
										if (no1Direct == 'U' || no1Direct == 'D') cout << "I" RESET;
										else cout << "-" RESET;
								}
								return true;
						}
				}

				if (Field.no1Place.size()) {
						for (int k = 0; k < Field.no1Place.size(); k++) {
								int no1X, no1Y;
								char shape;
								tie(no1X, no1Y, shape) = Field.no1Place[k];
								if (i == no1X && j == no1Y) {
										cout << BOLDGREEN << shape << RESET;
										return true;
								}
						}
				}

				return false;
		}

		bool printNo1AttackPoint(int i, int j, char tile, int& checkWarn, int playerAttackX, int playerAttackY) {
				if (Field.No1AttackPoint.size() == 0) return false;

				for (pair<int, int> point : Field.No1AttackPoint) {
						if (i == point.first && j == point.second) {
								cout << RED;
								if (i >= playerAttackX - 1 && i <= playerAttackX + 1 && j >= playerAttackY - 1 && j <= playerAttackY + 1) cout << MAGENTA;
								if (point.first == x && point.second == y) {
										cout << "o" << RESET;
										checkWarn = 1;
										return true;
								}
								cout << tile << RESET;
								return true;
						}
				}
				return false;
		}

		void drawNo1Body() {
				for (int k = 0; k < Field.No_1.size(); k++) {
						int no1X, no1Y;
						char no1Direct;
						tie(no1X, no1Y, no1Direct) = Field.No_1[k];
						if (no1Direct == 'U' || no1Direct == 'D') {
								if (grid[no1X][no1Y - 1] != '#') Field.no1Place.push_back({ no1X, no1Y - 1, 'o' });
								if (grid[no1X][no1Y + 1] != '#') Field.no1Place.push_back({ no1X, no1Y + 1, 'o' });

						}
						else if (no1Direct == 'L' || no1Direct == 'R') {
								if (grid[no1X + 1][no1Y] != '#') Field.no1Place.push_back({ no1X + 1, no1Y, 'o' });
								if (grid[no1X - 1][no1Y] != '#') Field.no1Place.push_back({ no1X - 1, no1Y, 'o' });
						}
				}
		}
};
class DepictMon {
public:
		bool printMon1(int i, int j, Mon& Mal, int playerAttackX, int playerAttackY) {
				for (array<int, 6> arr : Mal.vectorOfMon1) {
						if (i == arr[0] && j == arr[1]) {
								cout << RED;
								if (i >= playerAttackX - 1 && i <= playerAttackX + 1 && j >= playerAttackY - 1 && j <= playerAttackY + 1) cout << CYAN;
								cout << "a" RESET;
								return true;
						}
				}
				return false;
		}
};

DepictNo1 DNo1;
DepictMon DMon;

//출력
class ShowPartsOfMap {
		int numberOfBox;
public:

		void printCollapsedMap(int& splitFlag1, int fi, int fj, int nx1, int nx2, int ny1, int ny2, int add_length, char **matrix, int inDunFlag, Mon & Mal, int playerAttackX, int playerAttackY) {

				splitFlag1 = 4;
				if (printRow == INF) printRow = nx1;
				cout << "                  ";

				for (int printCol = ny1; printCol <= ny2; printCol++) {

						if (!inDunFlag && DMon.printMon1(printRow, printCol, Mal, playerAttackX, playerAttackY)) continue;

						int pbf = 0;
						if (directWall != -1) {
								pbf = wallCheck(printRow, printCol, add_length);
								if (pbf && !inDunFlag) {
										if (printRow == x && printCol == y) cout << RED "o" RESET;
										else cout << RED << matrix[printRow][printCol] << RESET;
								}
						}
						if (pbf && !inDunFlag) continue;

						if (isRun_M && M_AttackFlag && printRow >= atx1 && printRow <= atx2 && printCol >= aty1 && printCol <= aty2) {
								cout << RED;
								if (printRow >= playerAttackX - 1 && printRow <= playerAttackX + 1 && printCol >= playerAttackY - 1 && printCol <= playerAttackY + 1) cout << MAGENTA;
								if (printRow == mx && printCol == my) cout << "M" RESET;
								else if (printRow == x && printCol == y) cout << "o" RESET;
								else cout << matrix[printRow][printCol] << RESET;
						}
						else {
								if (!inDunFlag) {
										int disAbledValue = 0;
										if (DNo1.printNo1(printRow, printCol, playerAttackX, playerAttackY)) continue;
										if (DNo1.printNo1AttackPoint(printRow, printCol, matrix[printRow][printCol], disAbledValue, playerAttackX, playerAttackY)) continue;
								}

								if (isRun_M && !inDunFlag && printRow == mx && printCol == my) {
										cout << RED;
										if (printRow >= playerAttackX - 1 && printRow <= playerAttackX + 1 && printCol >= playerAttackY - 1 && printCol <= playerAttackY + 1) cout << CYAN;
										cout << "M" RESET;
								}
								else if (printRow == x && printCol == y) cout << CYAN "o" RESET;
								else if (matrix[printRow][printCol] == '^') {
										cout << GREEN;
										if (printRow >= playerAttackX - 1 && printRow <= playerAttackX + 1 && printCol >= playerAttackY - 1 && printCol <= playerAttackY + 1) cout << CYAN;
										cout << "^" RESET;
								}
								else if (matrix[printRow][printCol] == 's') {
										cout << BLUE;
										if (printRow >= playerAttackX - 1 && printRow <= playerAttackX + 1 && printCol >= playerAttackY - 1 && printCol <= playerAttackY + 1) cout << CYAN;
										cout << "s" RESET;
								}
								else {
										int hadDraw = 0;
										if (!inDunFlag) {
												for (int r = 0; r < countHabitat; r++) {
														int hx = habitatDictionary[r].first, hy = habitatDictionary[r].second;
														if (hx <= printRow && printRow <= hx + 2 && hy <= printCol && printCol <= hy + 2) {
																drawHabitat(r, matrix[printRow][printCol]);
																hadDraw = 1;
														}
												}
										}
										if (!hadDraw) {
												if (printRow >= playerAttackX - 1 && printRow <= playerAttackX + 1 && printCol >= playerAttackY - 1 && printCol <= playerAttackY + 1) cout << CYAN;
												cout << matrix[printRow][printCol] << RESET;
										}
								}
						}
				}

				if (printRow == nx1) {
						cout << "  --> 축소 시점";
						splitFlag1 = 6;
				}
				else if (printRow == (nx1 + nx2) / 2) {
						cout << "  mp : ";
						for (int i = 0; i < 3; i++) {
								if (i < mp) cout << BLUE "O" RESET;
								else cout << "O";
						}
						splitFlag1 = 5;
				}

				printRow++;

				return;
		}

		void printDistOfBox(int &splitFlag1, int &splitFlag2) {
				if (numberOfBox) cout << "                  상자까지 " << o_DistOfBox << "칸 남았습니다.";
				else cout << "                  남아있는 상자가 없습니다.";

				if (o_DistOfBox >= 10) splitFlag2 = 1; 
				else splitFlag2 = 2;

				if (!numberOfBox) splitFlag2 = 1;
				splitFlag1 = 1;
		}

		void printDistOfBoss(int &splitFlag1, int &splitFlag3) {
				cout << "                  M 까지 ";
				if (M_DistOfo >= 30) cout << BLUE << M_DistOfo << RESET;
				else if (22 < M_DistOfo && M_DistOfo < 30) cout << CYAN << M_DistOfo << RESET;
				else if (15 < M_DistOfo && M_DistOfo <= 22) cout << GREEN << M_DistOfo << RESET;
				else if (5 < M_DistOfo && M_DistOfo <= 15) cout << YELLOW << M_DistOfo << RESET;
				else  if (0 <= M_DistOfo && M_DistOfo <= 5) cout << RED << M_DistOfo << RESET;
				if (M_DistOfo != -1) cout << "칸 떨어져있습니다.";
				else cout << MAGENTA " 만나지 않습니다.  " RESET;
				if (M_DistOfo >= 100) splitFlag3 = 1;
				else if (M_DistOfo >= 10) splitFlag3 = 2;
				else splitFlag3 = 3;
				splitFlag1 = 2;
		}

		void printCircumstance(int nowc, int &splitFlag1, int& splitFlag2, int& splitFlag3, int fi, int i, int puM) {
				int quan, who;
				string what;
				tie(quan, what, who) = circum[nowc];

				int LengthOfString = (int) what.length();
				int alpha = 0;
				if (splitFlag1 == 1) {
						if (splitFlag2 == 1) cout << "     ";
						else if (splitFlag2 == 2) cout << "      ";
				}
				else if (splitFlag1 == 2) {
						if (splitFlag3 == 1) cout << "  ";
						else if (splitFlag3 == 2) cout << "   ";
						else if (splitFlag3 == 3) cout << "    ";
				}
				else if (splitFlag1 == 3) cout << "                     ";
				else if (splitFlag1 == 4) cout << "                   ";
				else if (splitFlag1 == 5) cout << "         ";
				else if (splitFlag1 == 6) cout << "    ";
				else cout << "                                                ";
				cout << "|";

				if (who == 1) {
						cout << CYAN "o" RESET << "가 상자에서 " << what << "을(를) " << quan << "개 획득했다.";
						alpha += 19;
				}
				else if (who == 2) {
						cout << RED "M" RESET << "이 상자에서 " << what << "을(를) " << quan << "개 획득했다.";
						alpha += 19;
				}
				else if (who == 3) {
						cout << CYAN "o" RESET << "가 " << what << "을(를) 섭취했다.";
						alpha += 14;
				}
				else if (who == 4) {
						cout << RED "M" RESET << "가 " << what << "을(를) 섭취했다.";
						alpha += 14;
				}
				else if (who == 5) {
						cout << CYAN "o" RESET << "가 공격을 받아 HP가 " RED << quan << RESET " 감소";
						alpha += 17;
				}
				else if (who == 6) {
						cout << RED << what << RESET << "(이)가 공격을 받아 HP가 " BLUE << quan << RESET " 감소";
						alpha += (int)what.size() + 16;
				}
				else if (who == 7) {
						if (what.length() == 4) {
								cout << CYAN "o" RESET << "의 " << what << "이 " << quan << "% 회복.";
						}
						else {
								cout << CYAN "o" RESET << "의 " << what << "이 " << quan << "% 증가.";
						}
						alpha += 10;
				}
				else if (who == 8) {
						if (what.length() == 4) {
								cout << RED "M" RESET << "의 " << what << "이 " << quan << "% 회복.";
						}
						else {
								cout << RED "M" RESET << "의 " << what << "이 " << quan << "% 증가.";
						}
						alpha += 10;
				}
				else if (who == -3) {
						if (quan) cout << "새로운 아군 " << BOLDCYAN << what << RESET;
						else cout << "새로운 아군 " << CYAN << what << RESET;
						alpha += 17;
				}
				else if (who == -4) {
						if (quan) cout << "새로운 적 " << BOLDRED << what << RESET;
						else cout << "새로운 적 " << RED << what << RESET;
						alpha += 17;
				}
				else if (who == -5) {
						cout << CYAN "o" RESET << "이 상점에서 " << what << "(을)를 " << quan << "개 구매";
						alpha += 17;
				}
				else if (who == -7) {
						cout << RED "M" RESET << "이 나타났다.";
						alpha += 8;
				}
				else if (who == -INF) {
						cout << RED << what << RESET << " 죽음";
						alpha += (int) what.size() + 3;
				}
				else if (who == -INF + 1) {
						cout << "get " << YELLOW << quan << RESET << "g";
				}
				else if (who == INF) {
						switch (quan) {
						case 0:
								cout << BOLDYELLOW "태초마을에 오신 것을 환영합니다" RESET;
								alpha += 17;
								break;
						case 1:
								cout << BOLDYELLOW "헛된 꿈" RESET;
								alpha += 4;
								break;
						}
				}
				else cout << what;

				if (i == fi - 1) {
						cout << " " << YELLOW "<- now" RESET;
						if (who) for (int i = 0; i < (90 - puM) + 42 - (LengthOfString + alpha); i++) cout << " ";
				}
				else if (who) for (int i = 0; i < (90 - puM) + 48 - (LengthOfString + alpha); i++) cout << " ";

		}
};
class Show {
		int MaxN, MaxM;
		int puN, puM;
		int numberOfBox;
		int nx1, nx2, ny1, ny2, mapTop, mapBottom, mapLeft, mapRight;
		int add_length;
		int si, sj;
		int fi, fj;
		int playerAttackX = -10, playerAttackY = -10;
public:
		int nowc, splitFlag1, splitFlag2, splitFlag3, beAttackedFlag;

		ShowPartsOfMap AccessoryWindow;
		
		void initializationParameters(int puN, int puM, int numberOfBox, int MaxN, int MaxM) {
				this->puN = puN, this->puM = puM;
				this->MaxN = MaxN, this->MaxM = MaxM;
				this->numberOfBox = numberOfBox;
				if (isAttackingOfo) nflag = 1;
				else nflag = 0;
				Field.no1Place.clear();
				add_length = 1;
				si = 0, sj = 0, fi = MaxN, fj = MaxM;
				nowc = 0, splitFlag1 = 0, splitFlag2 = 0, splitFlag3 = 0, beAttackedFlag = 0;
				playerAttackX = -1, playerAttackY = -1;
				tie(nx1, nx2, ny1, ny2) = monitor(x, y, 4, 5, MaxN, MaxM);
		}

		void placeAttackPoint() {
				if (M_AttackFlag) {
						tie(atx1, atx2, aty1, aty2) = attPoint(M_AttackDirect, mx, my, MaxN, MaxM);
						if (M_AttackDirect == 0 && atx2 > mx) atx2 = mx;
						else if (M_AttackDirect == 1 && atx1 < mx) atx1 = mx;
						else if (M_AttackDirect == 2 && aty2 > my) aty2 = my;
						else if (M_AttackDirect == 3 && atx2 < my) aty1 = my;

						beAttackedFlag = 1;
				}
		}

		void lengthOfBreak() {
				while (true) {
						if (add_length == 3) break;
						if (directWall == 0) {
								if (mx + add_length + 1 >= N) break;
								add_length++;
						}
						else if (directWall == 1) {
								if (mx - add_length - 1 < 0) break;
								add_length++;
						}
						else if (directWall == 2) {
								if (my + add_length + 1 >= M) break;
								add_length++;
						}
						else if (directWall == 3) {
								if (my - add_length - 1 < 0) break;
								add_length++;
						}
				}
		}

		void setSizeOfMap() {
				if (modMap == 1) tie(mapTop, mapBottom, mapLeft, mapRight) = monitor(x, y, 17, M / 2 - 1, MaxN, MaxM);
				else if (modMap == 2) tie(mapTop, mapBottom, mapLeft, mapRight) = monitor(x, y, N / 2 - 1, 43, MaxN, MaxM);
				else if (modMap == 3) tie(mapTop, mapBottom, mapLeft, mapRight) = monitor(x, y, 17, 43, MaxN, MaxM);

				if (modMap) si = mapTop, sj = mapLeft, fi = mapBottom + 1, fj = mapRight + 1;
		}

		void setSizeOfDun(int modDun) {
				if (modDun == 1) tie(mapTop, mapBottom, mapLeft, mapRight) = monitor(x, y, 17, MaxM / 2 - 1, MaxN, MaxM);
				else if (modDun == 2) tie(mapTop, mapBottom, mapLeft, mapRight) = monitor(x, y, MaxN / 2 - 1, 43, MaxN, MaxM);
				else if (modDun == 3) tie(mapTop, mapBottom, mapLeft, mapRight) = monitor(x, y, 17, 43, MaxN, MaxM);

				if (modDun) si = mapTop, sj = mapLeft, fi = mapBottom + 1, fj = mapRight + 1;
		}

		bool reduceSight(int i, int j) {
				for (int k = 0; k < T.sightPoint.size(); k++) {
						if (i == T.sightPoint[k].first && j == T.sightPoint[k].second) return false;
						else if (i < T.sightPoint[k].first) break;
				}
				if (i < x - 7 || i >= x + 7 || j < y - 12 || j >= y + 12) return true;
				return false;
		}

		void randomSight() {
				for (int i = x - 10; i <= x + 9; i++) {
						int randomV = rand() % 3;
						if (randomV == 1) T.sightPoint.push_back({ i, y - 15 });
						if (randomV == 2) T.sightPoint.push_back({ i, y + 14 });
						if (randomV == 1) T.sightPoint.push_back({ i, y - 14 });
						if (randomV == 2) T.sightPoint.push_back({ i, y + 13 });
						if (randomV == 1) T.sightPoint.push_back({ i, y - 13 });
						if (randomV == 2) T.sightPoint.push_back({ i, y + 12 });
				}
				for (int j = y - 15; j <= y + 14; j++) {
						int randomV = rand() % 3;
						if (randomV == 1) T.sightPoint.push_back({ x - 10, j });
						if (randomV == 2) T.sightPoint.push_back({ x + 9, j });
						if (randomV == 1) T.sightPoint.push_back({ x - 9, j });
						if (randomV == 2) T.sightPoint.push_back({ x + 8, j });
						if (randomV == 1) T.sightPoint.push_back({ x - 8, j });
						if (randomV == 2) T.sightPoint.push_back({ x + 7, j });
				}
				sort(T.sightPoint.begin(), T.sightPoint.end(), compare2);
		}

		void showPrint(char **matrix, int inDunFlag, Mon & Mal) {
				printRow = INF;

				if (Da.attackFlag) {
						pair<int, int> playerAttackPair = playerAttack();
						playerAttackX = playerAttackPair.first, playerAttackY = playerAttackPair.second;
				}

				DNo1.drawNo1Body();
				if (T.gameTimeNight == 1 && !inDunFlag) randomSight();

				for (int i = si; i < fi; i++) {
						splitFlag1 = 0, splitFlag2 = 0, splitFlag3 = 0;

						for (int j = sj; j < fj; j++) {
								int pbf = 0;

								if (T.gameTimeNight == 1 && !inDunFlag){
										if (reduceSight(i, j)) {
												cout << " ";
												continue;
										}
								}

								if (!inDunFlag && DMon.printMon1(i, j, Mal, playerAttackX, playerAttackY)) continue;

								if (directWall != -1) {
										pbf = wallCheck(i, j, add_length); //지금 좌표가 세 칸 짜리 벽 제거 부분에 속하는지 판별
										if (pbf) {
												if (!inDunFlag) {
														if (i == x && j == y) cout << RED "o" RESET;
														else cout << RED << matrix[i][j] << RESET;
												}
												deleteWall_list.push_back({ make_pair(i, j) });
										}
								}

								if (pbf && !inDunFlag) continue;

								if (isRun_M && M_AttackFlag && i >= atx1 && i <= atx2 && j >= aty1 && j <= aty2) { //beAttacked를 던전 내에서는 실행 안 하도록 하였기에 그냥 둬도 상관 없음
										cout << RED;
										if (i >= playerAttackX - 1 && i <= playerAttackX + 1 && j >= playerAttackY - 1 && j <= playerAttackY + 1) cout << MAGENTA;
										if (i == mx && j == my) cout << "M" RESET;
										else if (i == x && j == y) cout << "o" RESET;
										else cout << matrix[i][j] << RESET;
								}
								else {
										if (!inDunFlag) {
												int checkWarn = 0; //beAttackedFlag로 음악 내기 위해서 쓰는 변수
												if (DNo1.printNo1(i, j, playerAttackX, playerAttackY)) continue;
												if (DNo1.printNo1AttackPoint(i, j, matrix[i][j], checkWarn, playerAttackX, playerAttackY)) {
														if (checkWarn) beAttackedFlag = 1;
														continue;
												}
										}

										if (isRun_M && !inDunFlag && i == mx && j == my) {
												cout << RED;
												if (i >= playerAttackX - 1 && i <= playerAttackX + 1 && j >= playerAttackY - 1 && j <= playerAttackY + 1) cout << CYAN;
												cout << "M" RESET;
										}
										else if (i == x && j == y) {
												cout << CYAN "o" RESET;
										}
										else if (matrix[i][j] == '^') {
												cout << GREEN;
												if (i >= playerAttackX - 1 && i <= playerAttackX + 1 && j >= playerAttackY - 1 && j <= playerAttackY + 1) cout << CYAN;
												cout << "^" RESET;
										}
										else if (matrix[i][j] == 's') {
												cout << BLUE;
												if (i >= playerAttackX - 1 && i <= playerAttackX + 1 && j >= playerAttackY - 1 && j <= playerAttackY + 1) cout << CYAN;
												cout << "s" RESET;
										}
										else {
												int hadDraw = 0;
												if (!inDunFlag) {
														for (int k = 0; k < countHabitat; k++) {
																int hx = habitatDictionary[k].first, hy = habitatDictionary[k].second;
																if (hx <= i && i <= hx + 2 && hy <= j && j <= hy + 2) {
																		drawHabitat(k, matrix[i][j]);
																		hadDraw = 1;
																}
														}
												}
												if (!hadDraw) {
														if (i >= playerAttackX - 1 && i <= playerAttackX + 1 && j >= playerAttackY - 1 && j <= playerAttackY + 1) cout << CYAN;
														cout << matrix[i][j] << RESET;
												}
										}
								}
						}

						
						if (i + 4 == (i + fi) / 2 || printRow <= nx2) AccessoryWindow.printCollapsedMap(splitFlag1, fi, fj, nx1, nx2, ny1, ny2, add_length, matrix, inDunFlag, Mal, playerAttackX, playerAttackY);

						if (i == si) AccessoryWindow.printDistOfBox(splitFlag1, splitFlag2);
						else if (i == si + 1) AccessoryWindow.printDistOfBoss(splitFlag1, splitFlag3);
						else if (i == si + 2 && mp == 3){
										cout << "                  mp 가득참";
										splitFlag1 = 3;
						}
					
						if (nowc < circum.size()) AccessoryWindow.printCircumstance(nowc, splitFlag1, splitFlag2, splitFlag3, fi, i, puM);

						nowc++;

						cout << endl;
						
						

				}

				T.sightPoint.clear();
		}

		void backProcessing() {
				if (!nflag) {
						circum.erase(circum.begin() + 0);
						circum.push_back({ -1, "                                                 ", -1 });
				}
				if (beAttackedFlag) {
						thread soundThread(playSoundEffect, "C:\\Users\\happy\\Music\\MP_Answering Machine Beep.mp3", 12000000);
						soundThread.detach();
				}

				cout << "---";

				cout << endl << "현재 위치 (" << x << ", " << y << ")" << "     " << "현재 시간 ";

				if (T.gameTimeHours < 10) cout << 0;
				cout << T.gameTimeHours << " : ";
				if (T.gameTimeMinutes < 10) cout << 0;
				cout << T.gameTimeMinutes << "                              " << endl;
		}

};

//스탯 증가 관리
void increaseStat(const string key, const int who) {

		if (key == food_list[0]) {
				if (who == 7) p_alpha_at += 5;
				else if (who == 8) m_alpha_at += 5;
				circum.push_back({ 5, "공격력", who });
		}
		else if (key == food_list[1]) {
				if (who == 7) p_alpha_at += 10;
				else if (who == 8) m_alpha_at += 10;
				circum.push_back({ 10, "공격력", who });
		}
		else if (key == food_list[2]) {
				if (who == 7) p_alpha_de += 5;
				else if (who == 8) m_alpha_de += 5;
				circum.push_back({ 5, "방어력", who });
		}
		else if (key == food_list[3]) {
				if (who == 7) p_alpha_de += 10;
				else if (who == 8) m_alpha_de += 10;
				circum.push_back({ 10, "방어력", who });
		}
		else if (key == food_list[4]) {
				if (who == 7) p_alpha_hp += 10;
				else if (who == 8) m_alpha_hp += 10;
				circum.push_back({ 10, "최대 체력", who });
				circum.push_back({ 5, "체력", who });
				circum.erase(circum.begin() + 0);
		}
		circum.erase(circum.begin() + 0);

		playerAttackPoint = p_at + (p_at * (p_alpha_at / 100));
		playerDefensePoint = p_de + (p_de * (p_alpha_de / 100));
		playerMaxHealthPoint = max_p_hp + (max_p_hp * (p_alpha_hp / 100));

		MAttackPoint = m_at + (m_at * (m_alpha_at / 100));
		MDefensePoint = m_de + (m_de * (m_alpha_de / 100));
		MMaxHealthPoint = max_m_hp + (max_m_hp * (m_alpha_hp / 100));


		if (key == food_list[4]) {
				if (who == 7) playerHealthPoint += (playerMaxHealthPoint / 20);
				else if (who == 8) MHealthPoint += (MMaxHealthPoint / 20);
		}
}

//처맞을때 관련 
void shutDown(float AttackPoint, float DefensePoint, float& HealthPoint, const string by, const string who) {
		int typeWho;
		for (string character : nowAvailableCheck.enemy) {
				if (character == by) {
						typeWho = 5;
						break;
				}
		}
		for (string character : nowAvailableCheck.ally) {
				if (character == by) {
						typeWho = 6;
						break;
				}
		}

		if (by == "M") {
				thread soundThread(playSoundEffect, "C:\\Users\\happy\\Music\\MP_Shotgun 12ga Fire.mp3", 22000000);
				soundThread.detach();
				typeWho = 5;
		}
		else if (by == "No1") {
				thread soundThread(playSoundEffect, "C:\\Users\\happy\\Music\\MP_Decapitation.mp3", 17000000);
				soundThread.detach();
				typeWho = 5;
		}
		else if (by == "o") {
				thread soundThread(playSoundEffect, "C:\\Users\\happy\\Music\\997FA1385D28524F1D.mp3", 15000000);
				soundThread.detach();
				typeWho = 6;
		}

		int damage = (int)(AttackPoint - DefensePoint);
		if (damage < 0) damage = 0;

		circum.push_back({ damage, who, typeWho });
		circum.erase(circum.begin() + 0);
		HealthPoint -= (damage);
}
void shutDown2(float AttackPoint, float DefensePoint, int& HealthPoint, const string by, const string who) {
		thread soundThread(playSoundEffect, "C:\\Users\\happy\\Music\\997FA1385D28524F1D.mp3", 15000000);
		soundThread.detach();

		int damage = (int)(AttackPoint - DefensePoint);
		if (damage < 0) damage = 0;
		circum.push_back({ damage, who, 6 });
		circum.erase(circum.begin() + 0);
		HealthPoint -= (damage);
}

//던전
class InDun : public MakeMap {
		int numberOfBox;
		int modDun;
		bool isOn = false;
public:
		char** gridDun;
		int RowN, ColM;
		int startX, startY;
		Show PrintDun;

		InDun(int RowN, int ColM) {
				this->RowN = RowN;
				this->ColM = ColM;
				isOn = !isOn;
				gridDun = new char* [RowN];
				for (int i = 0; i < RowN; i++) gridDun[i] = new char[ColM];

				setMap(gridDun, RowN, ColM);

				pair<int, int> settingStartPair = setPlayerInDun(RowN, ColM);
				startX = settingStartPair.first, startY = settingStartPair.second;

				gridDun[startX][startY] = '@';

				if (RowN >= 35 && ColM >= 91) modDun = 3;
				else if (RowN >= 35) modDun = 1;
				else if (ColM >= 91) modDun = 2;

		}

		void runInDun(int puN, int puM, Mon & Mal) {
				int tmpPuN = puN, tmpPuM = puM;
				if (RowN < puN) tmpPuN = RowN;
				if (ColM < puM) tmpPuM = ColM;

				clearScreen();

				for (int i = 0; i < 36 - tmpPuN; i++) cout << endl;

				PrintDun.initializationParameters(tmpPuN, tmpPuM, -1, RowN, ColM);
				
				if (M_AttackFlag) PrintDun.placeAttackPoint();
				else if (directWall != -1) PrintDun.lengthOfBreak();

				PrintDun.setSizeOfDun(modDun);

				PrintDun.showPrint(gridDun, 1, Mal);

				PrintDun.backProcessing();
		}

		~InDun() {
				for (int i = 0; i < RowN; i++) delete[] gridDun[i];
				delete[] gridDun;
		}
};

//일반적인 게임 내 함수들
class Game {
		int gapEating;
		int numberOfBox;
		int nowDun = -1;
public:
		Show Print;
		InDun* numberDun[30];
		vector<int> onDun;
		int inDunFlag = 0;
		bool isMove = true;
		bool storeFlag = true;

		Game(int gapEating, int numberOfBox) { this->gapEating = gapEating, this->numberOfBox = numberOfBox; }

		void settingValueable(const int nx, const int ny, int &interval, MakeMap &Makemap) {
				x = nx, y = ny;

				nflag = 0, M_AttackFlag = 0;
				timer = time(NULL);

				if (!interval) {
						if (!isRun_M) {
								circum.push_back({ 1, "M", -7 });
								circum.erase(circum.begin());
								Makemap.setBoss();
								pair<int, int> bossLocation = Makemap.getBossLocation();
								mx = bossLocation.first, my = bossLocation.second;
								isRun_M = true;
						}
						else isRun_M = false;
						interval = appearanceInterval_M;
				}
		}

		void timeStream() {
				T.gameTimeMinutes++;
				if (T.gameTimeMinutes == 60) {
						T.gameTimeMinutes = 0;
						T.gameTimeHours++;
				}
				if (T.gameTimeHours == 24) {
						T.gameTimeHours = 0;
				}
				if (T.gameTimeHours < 6 || T.gameTimeHours >= 22) {
						T.gameTimeDaylight = 0, T.gameTimeNight = 1;
				}
				else if (T.gameTimeHours >= 6 || T.gameTimeHours < 12){
						T.gameTimeDaylight = 1, T.gameTimeNight = 0;
						thread soundThread(playSoundEffect, "C:\\Users\\happy\\Music\\MP_Bird In Rain.mp3", -50);
						soundThread.detach();
				}
		}

		char** nowPlayIn() {
				if (inDunFlag) return numberDun[nowDun]->gridDun;
				else return grid;
		}

		pair<int, int> setMaxSize() {
				int MaxN = N, MaxM = M;
				if (inDunFlag) MaxN = numberDun[nowDun]->RowN, MaxM = numberDun[nowDun]->ColM;

				return { MaxN, MaxM };
		}

		bool simulation(float sphp, float spat, float spde, float smhp, float smat, float smde) {

				int turnCount = 0;

				while (true) {

						if (turnCount > 100) return false;
						if (smhp <= 0) return false;

						if (sphp <= 0) return true;

						smhp -= (spat - smde);
						sphp -= (smat - spde);

						turnCount++;
				}
		}

		void findDistOfBox(Mon & Mal) {
				pair<int, int> maxSizePair = setMaxSize();
				int MaxN = maxSizePair.first, MaxM = maxSizePair.second;
				o_DistOfBox = moveBfs(x, y, -1, -1, 1, nowPlayIn(), MaxN, MaxM, Mal);
		}

		void M_Move(Mon & Mal) {
				int not_oFlag = 0;

				if (numberOfBox > 0 && !simulation(playerHealthPoint, playerAttackPoint, playerDefensePoint, MHealthPoint, MAttackPoint, MDefensePoint)) mmMod = 1;
				else mmMod = 0;

				if (breakWall == 1) not_oFlag = 1, breakWall = 0;

				if (!inDunFlag) M_DistOfo = moveBfs(mx, my, x, y, 2, grid, N, M, Mal);
				else M_DistOfo = -1;

				if (not_oFlag) breakWall = 1;

				if (M_DistOfo == -1) mmMod = 1;
				else {
						breakWall = 0;
						mmMod = 0;
				}

				if (!breakWall && mmMod) {
						if (moveBfs(mx, my, -1, -1, 2, grid, N, M, Mal) == -1) breakWall = 1;
						else breakWall = 0;
				}
				else if (breakWall) moveBfs(mx, my, -1, -1, 2, grid, N, M, Mal);

				if (grid[mx][my] == 'B') {
						getBox(mx, my, 2, numberOfBox);
						gapEating = 2;
				}
		}

		void M_Eat() {
				for (map<string, int>::iterator iter = M_foodDictionary.begin(); iter != M_foodDictionary.end(); iter++) {
						if (iter->second) {
								circum.push_back({ 1, iter->first, 4 });
								circum.erase(circum.begin());
								M_foodDictionary[iter->first]--;
								increaseStat(iter->first, 8);
								break;
						}
				}
		}

		void M_Action(Mon & Mal) {
				int sumItemCount = 0;

				if (gapEating) gapEating--;
				for (map<string, int>::iterator iter = M_foodDictionary.begin(); iter != M_foodDictionary.end(); iter++) {
						sumItemCount += iter->second;
				}
				int randomValue = rand() % 7;

				if (!randomValue && sumItemCount && !gapEating) M_Eat();
				else M_Move(Mal);

		}

		void beAttacked(Mon & Mal) {
				for (int i = 0; i < 4; i++) {
						int arc_nx = x + dx[i], arc_ny = y + dy[i];

						if (!nextCheck(N, M, arc_nx, arc_ny, ' ', -1, 0, 0, grid, Mal)) continue; //grid는 임시방편, 나중에 이 함수 매개변수에 따로 matrix 추가 할 것(안 할 수도 있음)

						if (arc_nx == mx && arc_ny == my) {
								M_AttackFlag = 1;
								M_AttackDirect = i;
								break;
						}
				}
		}

		void activity_M(Mon & Mal) {
				if (isRun_M) {
						findDistOfBox(Mal);
						M_Action(Mal);
						if (!inDunFlag) beAttacked(Mal);
						checkWall();
				}
				else M_DistOfo = -1;
		}

		void player_Attacking(Mon & Mal, __No__ & No__Event) {
				int oneTimeAttackNo1Flag = 0;
				pair<int, int> standardCoordination = playerAttack();
				int standardX = standardCoordination.first, standardY = standardCoordination.second;

				for (int i = standardX - 1; i <= standardX + 1; i++) {
						for (int j = standardY - 1; j <= standardY + 1; j++) {
								const int mob1Num = Mal.checkMob1Existence(i, j);
								const int isExist = No__Event.checkNo1Existence(i, j);
								if (!inDunFlag && isRun_M && i == mx && j == my) shutDown(playerAttackPoint, MDefensePoint, MHealthPoint, "o", "M");
								else if (!inDunFlag && isExist && !oneTimeAttackNo1Flag) {
										oneTimeAttackNo1Flag = 1;
										shutDown(playerAttackPoint, Field.No1_de, Field.No1_hp, "o", "털 없는 송충이");
								}
								else if (!inDunFlag && mob1Num != -1) {
										shutDown2(playerAttackPoint, stanCo.mon1DefensePoint, Mal.vectorOfMon1[mob1Num][4], "o", "동굴 벌레");
								}
						}
				}

		}

		void checkWall() {
				directWall = -1;
				if (breakWall) {
						for (int i = 0; i < 4; i++) {
								int nw_nx = mx + dx[i], nw_ny = my + dy[i];

								if (nw_nx < 0 || nw_ny < 0 || nw_nx >= N || nw_ny > M) continue;

								if (grid[nw_nx][nw_ny] == '#') {
										directWall = i;
										break;
								}
						}
				}
		}
		
		void checkDeleteWall(int nx, int ny) {
				if (M_AttackFlag && nx >= atx1 && nx <= atx2 && ny >= aty1 && ny <= aty2) shutDown(MAttackPoint, playerDefensePoint, playerHealthPoint, "M", "o");
				else if (directWall != -1) {
						for (int i = 0; i < deleteWall_list.size(); i++) {
								int br = deleteWall_list[i].first, bc = deleteWall_list[i].second;
								if (inDunFlag && nx == br && ny == bc) shutDown(MAttackPoint, playerDefensePoint, playerHealthPoint, "M", "o");
								else if (grid[br][bc] == '#') {
										grid[br][bc] = '.';
								}
						}
						if (inDunFlag) {
								thread soundThread(playSoundEffect, "C:\\Users\\happy\\Music\\MP_Shotgun 12ga Fire.mp3", 22000000);
								soundThread.detach();
						}
						deleteWall_list.clear();
				}
		}

		void No1Attacking(int nx, int ny) {
				int isAttackFlag = 0;
				for (pair<int, int> point : Field.No1AttackPoint) {
						if (grid[point.first][point.second] == '#') {
								grid[point.first][point.second] = '.';
						}
						else if (point.first == nx && point.second == ny) {
								isAttackFlag = 1;
								shutDown(Field.No1_at, playerDefensePoint, playerHealthPoint, "털 없는 송충이", "o");
						}
				}
				if (!isAttackFlag) {
						thread soundThread(playSoundEffect, "C:\\Users\\happy\\Music\\MP_Decapitation Head Fall Off.mp3", 29000000);
						soundThread.detach();
				}
				Field.No1AttackPoint.clear();
		}

		void printMap(const int puN, const int puM, int numberOfBox, Mon & Mal) {
				clearScreen();
				for (int i = 0; i < 36 - puN; i++) cout << endl;

				Print.initializationParameters(puN, puM, numberOfBox, N, M);

				if (M_AttackFlag) Print.placeAttackPoint();
				else if (directWall != -1) Print.lengthOfBreak();

				Print.setSizeOfMap();

				Print.showPrint(grid, 0, Mal);

				Print.backProcessing();
		}
		
		void prepareInDun(const int nx, const int ny, const int puN, const int puM) {
				inDunFlag = 1, isMove = false;
				nowDun = dictionaryOfDun[{nx, ny}];
				onDun.push_back(nowDun);
				if (!visitedDun[nowDun]) {
						thread soundThread(playSoundEffect, "C:\\Users\\happy\\Music\\MP_Robot Blip.mp3", 15000000);
						soundThread.detach();

						int randomValue1 = rand() % 3;
						int randomValue2 = rand() % 3 + 1;
						int newRowN = N / (randomValue1 + randomValue2), newColM = M / (randomValue1 + randomValue2);

						if (newRowN < 30) newRowN = 30;
						if (newColM < 40) newColM = 40;

						visitedDun[nowDun] = ~visitedDun[nowDun];
						numberDun[nowDun] = new InDun(newRowN, newColM);
				}
		}

		void printDun(int puN, int puM, Mon & Mal) { numberDun[nowDun] -> runInDun(puN, puM, Mal); }

		void pushItem(vector <pair<string, string> > &sellList, map<string, pair<int, string> > &itemDictionary, string element, string rarity, int price) {
				int randomValueG = rand() % 11;
				if (rarity == "Common") {
						sellList.push_back({ element, rarity });
						itemDictionary[element] = { price, rarity };
				}
				else if (rarity == "Uncommon") {
						if (randomValueG <= 9) {
								sellList.push_back({ element, rarity });
								itemDictionary[element] = { price, rarity };
						}
				}
				else if (rarity == "Rare") {
						if (randomValueG <= 7) {
								sellList.push_back({ element, rarity });
								itemDictionary[element] = { price, rarity };
						}

				}
				else if (rarity == "Execptional") {
						if (randomValueG <= 5) {
								sellList.push_back({ element, rarity });
								itemDictionary[element] = { price, rarity };
						}
				}
				else if (rarity == "Unique") {
						if (randomValueG <= 3) {
								sellList.push_back({ element, rarity });
								itemDictionary[element] = { price, rarity };
						}
				}
				else if (rarity == "Legendary") {
						if (randomValueG <= 2) {
								sellList.push_back({ element, rarity });
								itemDictionary[element] = { price, rarity };
						}
				}
				else if (rarity == "Myth") {
						if (randomValueG <= 1) {
								sellList.push_back({ element, rarity });
								itemDictionary[element] = { price, rarity };
						}
				}
				else if (rarity == "Real") {
						if (randomValueG == 0) {
								sellList.push_back({ element, rarity });
								itemDictionary[element] = { price, rarity };
						}
				}

		}

		void enterStore(int puN, int puM, const int nx, const int ny, Achievements &Wow) {
				int noExistFlag = 0, toMuchFlag = 0, soCheapFlag = 0;
				long long noExistFlag2 = 0;

				vector<pair<string, string> > sellingListFood;
				vector<pair<string, string> > sellingListIngredient;
				vector<pair<string, string> > sellingListWeapon;
				vector<pair<string, string> > sellingListArmor;
				map<string, pair<int, string> > itemDictionary;

				for (tuple<string, string, int> Food : standard.availableSellFood) {
						string food, rarity;
						int price;
						tie(food, rarity, price) = Food;
						pushItem(sellingListFood, itemDictionary, food, rarity, price);
				}
				for (tuple<string, string, int> Ingredient : standard.availableSellIngredient) {
						string ingredient, rarity;
						int price;
						tie(ingredient, rarity, price) = Ingredient;
						pushItem(sellingListIngredient, itemDictionary, ingredient, rarity, price);
				}
				for (tuple<string, string, int> Weapon : standard.availableSellWeapon) {
						string weapon, rarity;
						int price;
						tie(weapon, rarity, price) = Weapon;
						pushItem(sellingListWeapon, itemDictionary, weapon, rarity, price);
				}
				for (tuple<string, string, int> Armor : standard.availableSellArmor) {
						string armor, rarity;
						int price;
						tie(armor, rarity, price) = Armor;
						pushItem(sellingListArmor, itemDictionary, armor, rarity, price);
				}
				
				if(storeFlag) cin.ignore();
				storeFlag = false;
				while (true) {
						system("cls");
						cout << endl;
						for (int i = 0; i < (puM + 60) / 2; i++) cout << "_";
						cout << YELLOW " 상 점 " RESET;
						for (int i = 0; i < (puM + 60) / 2; i++) cout << "_";
						cout << "_" << endl << endl;


						int Num = 0;
						while (Num < sellingListFood.size() || Num < sellingListIngredient.size() || Num < sellingListWeapon.size() || Num < sellingListArmor.size()) {
								int alpha;
								if (Num < sellingListFood.size()) {
										alpha = 30 - (int) sellingListFood[Num].first.size();
										cout << sellingListFood[Num].first << " : " << itemDictionary[sellingListFood[Num].first].first << "g";
								}
								for (int i = 0; i < alpha; i++) cout << " ";

								if (Num < sellingListIngredient.size()) {
										alpha = 30 - (int) sellingListIngredient[Num].first.size();
										cout << sellingListIngredient[Num].first << " : " << itemDictionary[sellingListIngredient[Num].first].first << "g";
								}
								for (int i = 0; i < alpha; i++) cout << " ";

								if (Num < sellingListWeapon.size()) {
										alpha = 30 - (int) sellingListWeapon[Num].first.size();
										cout << sellingListWeapon[Num].first << " : " << itemDictionary[sellingListWeapon[Num].first].first << "g";
								}
								for (int i = 0; i < alpha; i++) cout << " ";

								if (Num < sellingListArmor.size()) cout << sellingListArmor[Num].first << " : " << itemDictionary[sellingListArmor[Num].first].first << "g";

								Num++;
								cout << endl;
						}
						for (int i = 0; i < 40 - (Num * 2); i++) cout << endl;
						for (int i = 0; i < (puM + 60) / 2; i++) cout << "_";
						cout << "_________";
						for (int i = 0; i < (puM + 60) / 2; i++) cout << "_";
						cout << endl << "현재 보유하고 있는 돈 : " << YELLOW << money << RESET << "g" << endl;

						string toBuy;
						int quan;

						if (noExistFlag) cout << endl << RED "잘못된 입력" RESET;
						else if (noExistFlag2) {
								cout << endl;
								if (money * 100000 <= noExistFlag2) {
										cout << BOLDYELLOW "사기엔 돈이 너무 x 1e5 부족하다." RESET;
										Wow.achievements2();
								}
								else if (money * 5 <= noExistFlag2) cout << RED "사기엔 돈이 너무너무너무너무너무 부족하다." RESET;
								else if (money * 3 <= noExistFlag2) cout << RED "사기엔 돈이 너무너무너무 부족하다." RESET;
								else if (money * 2 <= noExistFlag2) cout << RED "사기엔 돈이 너무너무 부족하다." RESET;
								else cout << RED "사기엔 돈이 부족하다." RESET;
						}
						else if (toMuchFlag) cout << endl << RED "개수는 1000개를 넘을 수 없습니다." RESET;
						else if (soCheapFlag) cout << endl << RED "1개 미만으로 구매하실 수는 없습니다." RESET;
 						else cout << endl;

						noExistFlag = 0, noExistFlag2 = 0, toMuchFlag = 0, soCheapFlag = 0;

						cout << endl << "구매할 아이템 : ";
						getline(cin, toBuy);

						if (toBuy != "0") {
								if (!itemDictionary.count(toBuy)) {
										noExistFlag = 1;
										continue;
								}
						}
						else {
								grid[nx][ny] = '^', grid[nx - 1][ny] = '.', grid[nx - 1][ny + 1] = '.', grid[nx][ny + 1] = '.';
								system("cls");
								return;
						}

						cout << endl << "수량 : ";
						cin >> quan;

						if (cin.fail()) {
								cin.clear();
								noExistFlag = 0;
								thread soundThread(playSoundEffect, "C:\\Users\\happy\\Music\\MP_Large Metal Pan 2.mp3", 15000000);
								soundThread.detach();
								continue;
						}
						else {
								if (quan >= 1001) {
										toMuchFlag = 1;
								}
								else if (quan <= 0) {
										soCheapFlag = 1;
								}
								else if (itemDictionary[toBuy].first * quan > money) {
										noExistFlag2 = itemDictionary[toBuy].first * quan;
   							}
								else {
										money -= itemDictionary[toBuy].first * quan;
										circum.push_back({ quan, toBuy, -5 });
										circum.erase(circum.begin());
								}
								cin.ignore();
						}

				}
		}

		pair<int, int> eventRun(char **matrix, const int nx, const int ny, int puN, int puM, Achievements &Wow) {
				if (nx == 0 && ny == 0 && !inDunFlag) Wow.achievements1();
				if (matrix[nx][ny] == '^' && mp < 3) mp++;
				else if (matrix[nx][ny] == 'B') getBox(nx, ny, 1, numberOfBox);
				else if (matrix[nx][ny] == 'P') enterStore(puN, puM, nx, ny, Wow);
				else if (isMove && matrix[nx][ny] == '@') {
						system("cls");
						if (!inDunFlag) {
								tmpNx = nx, tmpNy = ny;
								prepareInDun(nx, ny, puN, puM);

								return { numberDun[nowDun]->startX, numberDun[nowDun]->startY };
						}
						else {
								inDunFlag = 0;
								return { tmpNx, tmpNy };
						}
				}

				return{ nx, ny };
		}
		
		~Game() {
				for (int i : onDun){
						if (numberDun[i] != nullptr) {
								delete numberDun[i];
								numberDun[i] = nullptr;
						}
				}
		}

};

//플레이어 옵션
class CheckInput {
public:
		char attackOptions() {
				Da.attackFlag = 1;

				while (true) {
						char selectDirect = _getch();

						switch (selectDirect) {
						case '0':
								return '0';
								break;
						case 119:
								return 'w';
								break;
						case 97:
								return 'a';
								break;
						case 100:
								return 'd';
								break;
						case 115:
								return 's';
								break;
						case 13:
								if(Da.frontA == 1 || Da.leftA == 1 || Da.rightA == 1 || Da.downA == 1) return '1';
						}
				}
		}

		void enemy_status(string figure) {
				cout << endl;
				cout << figure << endl << endl;
				if (figure == "M") {
						cout << "HP : " << MHealthPoint << "/" << MMaxHealthPoint << endl;
						cout << "att : " << MAttackPoint << endl;
						cout << "def : " << MDefensePoint << endl;
						cout << endl << "지성체(" << BOLDCYAN "passive" RESET << ") : 특수적 행동이 가능" << endl;
						cout << endl << "속전속결(" << BOLDCYAN "passive" RESET << ") : 공격 이후의 간격이 감소" << endl;
						cout << endl << "강타(" << BOLDRED "active" RESET << ") : 전방 공격, 범위는 위력에 비례한다." << endl;
						setCursor(30, 32);

						cout << RED "M" RESET;
				}
				else if (figure == "털 없는 송충이") {
						cout << "HP : " << Field.No1_hp << "/" << Field.max_No1_hp << endl;
						cout << "att : " << Field.No1_at << endl;
						cout << "def : " << Field.No1_de << endl;
						cout << endl << "변이(" << BOLDCYAN "passive" RESET << ") : 거대화, 신체적 능력 상승, 수중에서 체력이 지속적으로 회복" << endl;
						cout << endl << "성충 기간(" << BOLDCYAN "passive" RESET << ") : 혈액을 갈망" << endl;
						cout << endl << "물기(" << BOLDRED "active" RESET << ") : 넓은 지역을 공격" << endl;
						setCursor(30, 30);

						for (int i = 0; i <= 25; i++) cout << BOLDGREEN "o" << RESET;
						setCursor(30, 31);
						for (int i = 25; i >= 0; i--) {
								if (i == 0) cout << BOLDRED "O" RESET;
								else if ((2 <= i && i <= 5) || (13 <= i && i <= 16) || (20 <= i && i <= 23)) cout << RED "o" RESET;
								else cout << RED "-" RESET;
						}
						setCursor(30, 32);
						for (int i = 0; i <= 25; i++) cout << BOLDGREEN "o" << RESET;
				}
				else if (figure == "동굴 벌레") {

				}
				cout << endl;
		}

		void ally_status(string figure) {
				cout << endl;
				cout << figure << endl << endl;
				if (figure == "o") {
						cout << "HP : " << playerHealthPoint << "/" << playerMaxHealthPoint << endl;
						cout << "att : " << playerAttackPoint << endl;
						cout << "def : " << playerDefensePoint << endl;
						setCursor(30, 30);

						cout << CYAN "o" RESET;
				}


		}

		void checkItem(string figure) {
				cout << endl;
				cout << figure << endl << endl;

				if (figure == "썩은 닭가슴살") {
						cout << "공격력이 5% 상승한다. (x + (x / 20))" << endl << endl;
						cout << "너무 오래 구운 것을 포함하는 명칭이다.";
				}
				else if (figure == "맛있는 닭가슴살") {
						cout << "공격력이 10% 상승한다. (x + (x / 10))" << endl << endl;
						cout << "적당히 구운 닭가슴살이다. 느끼하지 않다.";
				}
				else if (figure == "중국산 찹쌀약과") {
						cout << "방어력이 5% 상승한다. (x + (x / 20))" << endl << endl;
						cout << "크기도 별로. 그냥 맛없다는 것이 특징이다.";
				}
				else if (figure == "국내산 찹쌀약과") {
						cout << "방어력이 10% 상승한다. (x + (x / 10))" << endl << endl;
						cout << "원래 그나마 맛있는 것은 국내산으로 통칭한다.";
				}
				else if (figure == "강황밥") {
						cout << "최대체력이 10% 상승한다. (x + (x / 10))" << endl;
						cout << "체력을 5% 회복한다. (x + (x / 20))" << endl << endl;
						cout << "효능이 좋은 음식" << endl;
						cout << "씨앗을 구하는 것이 어렵기 때문에 재배지역은 한정적이다.";
				}
		}

		bool useItem(string figure) {
				if (food_dict.count(figure)) {
						if (food_dict[figure] == 0) {
								thread soundThread(playSoundEffect, "C:\\Users\\happy\\Music\\MP_Large Metal Pan 2.mp3", 15000000);
								soundThread.detach();
								return false;
						}

						thread soundThread(playSoundEffect, "C:\\Users\\happy\\Music\\995573385CFFD24F07.mp3", 18000000);
						soundThread.detach();
						food_dict[figure]--;

						circum.push_back({ 1, figure, 3 });
						circum.erase(circum.begin() + 0);
						increaseStat(figure, 7);

						return true;
				}
				else {

				}
				return false;
		}

		int mainCheckInput(vector<string> optionList) {
				int nowNum = 0;
				int sumNum = (int)optionList.size();
				string figure = " ";

				while (true) {
						int cinFinish = 0;

						system("cls");
						setCursor(30, 30);
						cout << YELLOW "  Options" RESET << endl << endl << "--" << endl << endl;

						for (int i = 0; i < sumNum; i++) {
								if (nowNum == i) cout << YELLOW;
								cout << optionList[i] << "   " << RESET;
						}

						figure = optionList[nowNum];

						char selectCheck = _getch();

						switch (selectCheck) {
						case '0':
								cinFinish = 1;
								break;
						case 119: //w
								break;
						case 97: //a
								if ((nowNum - 1) % 10 < 0) continue;
								nowNum--;
								break;
						case 100: //d
								if (nowNum + 1 >= sumNum) continue;
								nowNum += 1;
								break;
						case 115: //s
								break;
						case 13:
								return nowNum + 1;
						}
						if (cinFinish) break;
				}

				system("cls");
				return 0;
		}

		int subCheckInput(int idx, int numberOfBox, const vector<string>& List) {
				int nowNum = 0;
				string figure = " ";

				while (true) {
						int cinFinish = 0;

						system("cls");
						cout << endl << endl;
						if (idx == 2)	cout << YELLOW "  Inventory" RESET << endl << endl << "--" << endl << endl;
						else cout << YELLOW "  Search" RESET << endl << endl << "--" << endl << endl;

						int sumNum = (int)List.size();
						for (int i = 0; i < 10; i++) {
								cout << "  ";
								for (int j = 0; j <= sumNum / 10; j++) {
										int num = i + (10 * j);
										if (num >= sumNum) break;

										if (nowNum == num) {
												if (idx == 2) cout << YELLOW << List[num] << " X " << food_dict[List[num]] << RESET;
												else cout << YELLOW << List[num] << RESET;
										}
										else {
												if (idx == 2) {
														if (food_dict[List[num]] == 0) cout << GRAY;
														cout << List[num] << " X " << food_dict[List[num]];
														cout << RESET;
												}
												else cout << List[num];
										}

										int alpha = 25 - (int)List[num].size();
										for (int k = 0; k < alpha; k++) cout << " ";
								}
								cout << endl << endl;
						}
						cout << "--" << endl << endl;

						if (sumNum) figure = List[nowNum];

						if (idx == 0) ally_status(figure);
						else if (idx == 1) enemy_status(figure);
						else if (idx == 2) checkItem(figure);

						char selectCheck = _getch();

						switch (selectCheck) {
						case '0':
								cinFinish = 1;
								break;
						case 119: //w
								if ((nowNum - 1) % 10 < 0) continue;
								nowNum--;
								break;
						case 97: //a
								if (nowNum - 10 < 0) continue;
								nowNum -= 10;
								break;
						case 100: //d
								if (nowNum + 10 >= sumNum) continue;
								nowNum += 10;
								break;
						case 115: //s
								if (nowNum + 1 >= sumNum) continue;
								nowNum += 1;
								break;
						case 13: //enter
								useItem(figure);
						}
						if (cinFinish) break;
				}

				system("cls");
				return -1;
		}
};

//플레이어 죽을 때
void death_que() {
		for (int i = 0; i < 5; i++) {
				cout << "." << endl;
				this_thread::sleep_for(chrono::milliseconds(500));
		}
		cout << "죽었다" << endl;
		this_thread::sleep_for(chrono::milliseconds(1200));
}

//런타임
int main() {
		Start FirstWork;
		MakeMap Makemap;

		FirstWork.setEnvironment();
		FirstWork.input();
		int puN = FirstWork.getPuN();
		int puM = FirstWork.getPuM();
		FirstWork.firstCircumPush();
		FirstWork.setFirstAvailableFood();
		FirstWork.setFirstAvailableOptions();

		Makemap.assignMapArray();
		Makemap.setMap(grid, N, M);
		Makemap.setQuan();
		Makemap.placeLabyrinth();
		Makemap.setPlayerInMap();
		Makemap.setBoss();
		Makemap.placeBox();
		Makemap.placeStore();
		Makemap.setBossHabitat();

		pair<int, int> playerLocation = Makemap.getPlayerLocation(), bossLocation = Makemap.getBossLocation();
		x = playerLocation.first, y = playerLocation.second;
		mx = bossLocation.first, my = bossLocation.second;
		int numberOfBox = Makemap.getNumberOfBox();

		int nx = x, ny = y;
		mp = 0, M_mp = 0;
		system("cls");

		Game Run(0, numberOfBox);
		CheckInput PrimeS;
		__No__ No__Event;
		Mon Mal;
		Achievements Wow;
		Wow.setList();

		int appearanceIntervalPeriod_M = appearanceInterval_M; 

		while (true) {
				Run.settingValueable(nx, ny, appearanceIntervalPeriod_M, Makemap);
				Run.timeStream();

				Run.activity_M(Mal);

				if(Field.No1AttackPoint.size() == 0 && !Run.inDunFlag) No__Event.No_1Run();

				if(Mal.intervalMon1()) Mal.createMon1();
				Mal.activeMon();

				if (!Run.inDunFlag) Run.printMap(puN, puM, numberOfBox, Mal);
				else Run.printDun(puN, puM, Mal);


				if (deathFlag) {
						death_que();
						break;
				}

				char selectNext;
				int cinf = 0;
				Run.isMove = true;

				while (true) {
						int failCount = 0;

						selectNext = _getch();

						switch (selectNext) {
						case 119:
								nx = x - 1, ny = y;
								break;
						case 97:
								nx = x, ny = y - 1;
								break;
						case 100:
								nx = x, ny = y + 1;
								break;
						case 115:
								nx = x + 1, ny = y;
								break;
						case 50:
								nx = x - 2, ny = y;
								break;
						case 113:
								nx = x, ny = y - 2;
								break;
						case 101:
								nx = x, ny = y + 2;
								break;
						case 120:
								nx = x + 2, ny = y;
								break;
						case 112:
								nx = x, ny = y, Run.isMove = false;
								break;
						case 103:
								isAttackingOfo = true;
								while (true) {
										char attDirect = PrimeS.attackOptions();
										if(attDirect != '1') Da.frontA = 0, Da.leftA = 0, Da.rightA = 0, Da.downA = 0;

										if (attDirect == '0') break;
										else if (attDirect == 'w') Da.frontA = 1;
										else if (attDirect == 'a') Da.leftA = 1;
										else if (attDirect == 's') Da.downA = 1;
										else if (attDirect == 'd') Da.rightA = 1;
										else if (attDirect == '1') {
												Run.player_Attacking(Mal, No__Event);
												Da.frontA = 0, Da.leftA = 0, Da.rightA = 0, Da.downA = 0;
												Mal.checkDeathMon1();
												break;
										}

										if (!Run.inDunFlag) Run.printMap(puN, puM, numberOfBox, Mal);
										else Run.printDun(puN, puM, Mal);

								}
								isAttackingOfo = false;
								break;
						case 111:
								int options;
								Run.storeFlag = true;

								while (true) {
										options = PrimeS.mainCheckInput(nowAvailableCheck.option);

										if (!options) cinf = 1;
										else if (options == 1) {
												PrimeS.subCheckInput(0, numberOfBox, nowAvailableCheck.ally);
												continue;
										}
										else if (options == 2) {
												PrimeS.subCheckInput(1, numberOfBox, nowAvailableCheck.enemy);
												continue;
										}
										else if (options == 3) {
												PrimeS.subCheckInput(2, numberOfBox, nowAvailableCheck.item);
												continue;
										}
										break;
								}

								if (cinf) {
										cinf--;
										if (!Run.inDunFlag) Run.printMap(puN, puM, numberOfBox, Mal);
										else Run.printDun(puN, puM, Mal);
										continue;
								}
								break;
						default:
								thread soundThread(playSoundEffect, "C:\\Users\\happy\\Music\\MP_Large Metal Pan 2.mp3", 15000000);
								soundThread.detach();
								continue;
						}

						if (!nextCheck(Run.setMaxSize().first, Run.setMaxSize().second, nx, ny, selectNext, -1, 0, 0, Run.nowPlayIn(), Mal)) continue;
						if (selectNext == '2' || selectNext == 'q' || selectNext == 'e' || selectNext == 'x') {
								if (mp >= 2) mp -= 2;
								else {
										thread soundThread(playSoundEffect, "C:\\Users\\happy\\Music\\MP_Computer Error.mp3", 15000000);
										soundThread.detach();
										continue;
								}
						}

						break;
				}

				Run.checkDeleteWall(nx, ny);
				if(Field.No1AttackPoint.size() > 0) Run.No1Attacking(nx, ny);

				if (playerHealthPoint <= 0) deathFlag = 1;

				pair<int, int> changeCoordinates= Run.eventRun(Run.nowPlayIn(), nx, ny, puN, puM, Wow);
				nx = changeCoordinates.first, ny = changeCoordinates.second;

				appearanceIntervalPeriod_M--;
		}

		grid = nullptr;

		return 0;
}
