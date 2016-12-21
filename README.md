#include <iostream>
#include <stdio.h>
#include <windows.h>
#include <cstring>
#include <ctime>
#include <time.h>
#include <stdlib.h>
#include <string>
#include <conio.h>
#include <list>
#include <cstdlib>
#include <Turboc.h>
#include <cassert>
#include <algorithm> //transform 사용
#include <functional> //bind1st
#define SIZE 100

const int MaxCard=52;
const int GetCard=5;
const int VailCard=6;
const int CardGap=4;
const int TotalCard=10;
const int Speed=1000;
const int PromptSpeed=2000;
int inputKey;
char a;


using namespace std;

void gotoXY(int x, int y) 
{ 
   COORD pos={x,y};
    SetConsoleCursorPosition(GetStdHandle(STD_OUTPUT_HANDLE),pos);  
}

struct SCard  // 카드 한장한장 설정해주는것. 즉 카드 하나하나 분별해주는 구조체   이 클래스는 카드의 '속성'에 대해 관리하는 ㅜ조체 
{
   char Name[4];
   SCard() { Name[0] == 0; }
   SCard(const char *pName) { strcpy(Name,pName); }

   string getName() {
      return string(this->Name);
   }

   void setName(char *name) {
      strcpy(Name, name);
   }
   
   void setName(string name) {
      strcpy(Name, name.c_str());
   }

   int GetNumber() const {
      if(isdigit(Name[0])) return Name[0]-'0';
      if(Name[0] =='J') return 11;
      if(Name[0] =='Q') return 12;
      if(Name[0] =='K') return 13;
      if(Name[0] =='A') return 1;
      if(Name[0] =='T') return 10;

   };

   char GetKind() const{
      if(strcmp(Name+1,"♠")==0 ) return 'S';
      else if(strcmp(Name+1,"◆")==0 ) return 'D';
      else if(strcmp(Name+1,"♣")==0 ) return 'C';
      else if(strcmp(Name+1,"♥")==0 ) return 'H';
   }

   friend ostream &operator <<(ostream &c, const SCard &C)
   {
      return c << C.Name;
   }

   bool operator ==(const SCard & Other) const {
      return (strcmp(Name,Other.Name) == 0);
   }

   bool operator <(const SCard &Other) const {
      if( GetNumber() < Other.GetNumber()) return true;
      if( GetNumber() > Other.GetNumber()) return false;
      if( GetKind() < Other.GetNumber()) return true;
      return false;
   }
};

SCard Trump[MaxCard]={
   "A♠","2♠","3♠","4♠","5♠","6♠","7♠","8♠","9♠","T♠","J♠","Q♠","K♠",
   "A◆","2◆","3◆","4◆","5◆","6◆","7◆","8◆","9◆","T◆","J◆","Q◆","K◆",
   "A♣","2♣","3♣","4♣","5♣","6♣","7♣","8♣","9♣","T♣","J♣","Q♣","K♣",
   "A♥","2♥","3♥","4♥","5♥","6♥","7♥","8♥","9♥","T♥","J♥","Q♥","K♥"
};

class CCardSet : public SCard // 이 클래스는 카드들의 '상태 또는 출력'을 맡는 클래스 
{
   private:
   
   protected:
      SCard MTrump[GetCard];
      SCard YTrump[GetCard];
   
   public:
   CCardSet();
   SCard *getMTrump() { return MTrump; }  // 이쪽부분 MTrump 반환해주는 부분
   SCard *getYTrump() { return YTrump; }  // 이쪽부분 RTrump 반환해주는 부분
   void randomCard();
   void showCard();
   void judgeWin();
   void AllCard();
};

CCardSet::CCardSet()
{
   int i;
   
   for(i=0;i<GetCard;i++)
   {
      MTrump[i]="???"; // CTrump은 현재 아무것도 들어가지 않은상태 
      YTrump[i]="???";
   }
}

void CCardSet::randomCard()
{
   int i,bFound,j;
   int a[10]; // 중복확인 해주기 위한 소스 

   for ( i = 0; i < TotalCard; i++ )
       {
             while (1)
             {
                a[i] = rand() % 52;
                bFound = 0;
                
                for ( j = 0; j < i; ++j )
                {         
                           if ( a[j] == a[i] )
                           {
                                 bFound = 1;
                                 break;
                           }
                    }
                    if ( !bFound )
                           break;
             }
            if(i<5)
            {
               MTrump[i]=Trump[a[i]];
            } 
            else if(i>=5)
            {
               YTrump[i-5]=Trump[a[i]];
            }
       }
       
}

void CCardSet::showCard()
{
   int i;
   gotoxy(10,10);
   for(i=0;i<3;i++)
   {
         if(i == 0){
            cout << "상대방 카드: " ;
         }
         cout << YTrump[i]<< "   ";
   }
   for(i=0;i<2;i++)
   {
      cout << " ??? ";
   }
   cout << "\n\n\n\n";
   gotoxy(10,30);
   for(i=0;i<3;i++)
   {
      if (i == 0)
      {
         cout << " 나의  카드: ";
     }
      cout << MTrump[i] << "   " ;
   }
   for(i=0;i<2;i++)
   {
      cout << " ??? ";
   }
}

void CCardSet::AllCard()
{
   int i;
   gotoxy(10,10);
   for(i=0;i<GetCard;i++)
   {
         if(i == 0){
            cout << "상대방 카드: " ;
         }
         cout << YTrump[i]<< "   ";
   }

   cout << "\n\n\n\n";
   gotoxy(10,30);
   for(i=0;i<GetCard;i++)
   {
      if (i == 0)
      {
         cout << " 나의  카드: ";
     }
      cout << MTrump[i] << "   " ;
   }

}





class CPlayer : public SCard
{
   private:
      string Name;
      int ScoreCard;
      int Life;

    protected:
       SCard CopyTrump[GetCard];

    public:
      CPlayer();
      
      void GetLostlife(int check,int life);
      
      void ShowLife();
      int Showscore();
      void copy(SCard trump[]);
      void ShowCopyTrump();
      int CalCard(); // 자기 카드의 총점을 계산해주는 함수 
      int Betting();
      
};

CPlayer::CPlayer()
{
   Life=100;
}

void CPlayer::ShowLife()
{
   cout << Life;
}

void CPlayer::GetLostlife(int check,int life)
{
   if(check == 1)  // 게임에서 승리를 하게된다면 혹은 Y/N 에서 Y을 눌렀을 경우
   {
      Life+=life;   
      
   } 
   
   else if(check == 0) // 게임에서 패배할 경우  혹은 Y/N 에서 N을 눌렀을 경우 
   {
      Life-=life;      
   }
}

int CPlayer::Showscore()
{
   return ScoreCard;
}

int CPlayer::Betting()  //라이프 입력칸 
{
   int life;
   cin >> life;
   
   if(Life < life)
   {
      cout << "라이프가 부족합니다." << endl;
   }
   else
        Life -=life;
        
      return life;
   
}

void CPlayer::copy(SCard trump[])
{
   for(int i=0; i<GetCard; i++) 
   {
      CopyTrump[i].setName(trump[i].getName());
   }
}

void CPlayer::ShowCopyTrump()
{
   for(int i=0;i<GetCard;i++)
   {
      cout << CopyTrump[i];
   }
}

int CPlayer::CalCard()
{
   for(int i=0;i<GetCard;i++)
   {
      ScoreCard+=CopyTrump[i].GetNumber();
   }
   return ScoreCard;
}




class Help /*도움말 */ 
{
   public:
      void ShowHelp()
      {
            system("cls");
            cout << "게임의 도움말 입니다." << endl;
            cout << " " << endl;
            cout << "생존점수의 차이가 9배가 날때까지 게임을 합니다. (20점대 180점이 면 패배)" << endl;
            cout << " " << endl;
            cout << "게임이 시작되면 5장의 카드를 받습니다." << endl;
            cout << " " << endl;            
            cout << "받은 카드를 확인하고, 동시에 1장씩 공개합니다. 카드는 3장까지 공개합니다." << endl;
            cout << " " << endl;
         cout << "여기서 플레이어는 제안을 할수  있습니다." << endl;
            cout << " " << endl;
         cout << "3장째에서 배팅을 시작합니다." << endl;
            cout << " " << endl;
         cout << "배팅 결과가 끝나면 판에 펼쳐진 카드 5장의 총합을 계산하고, 서로 비교합니다." << endl;
            cout << " " << endl;
         cout << "카드의 총합이 높은사람이 1승을 거두게 됩니다." << endl;
            cout << " " << endl;
         cout << "이렇게 1번의 게임이 종료 됩니다." << endl;
         cout << " " << endl;

            getch();
         a=getch();
         cin >> a;

      }
};

class Exit /* 프로그램 종료 */ 
{
   public:
      void TheExit()
      {
         gotoXY(50,25);
          system("cls");
         cout <<" 게임을 잠시후 종료합니다!";
         getch();
         Sleep(2000);
         exit(1);
      }
};

int main()
{
   system("mode con:cols=130 lines=40");
   CCardSet Set;
   CPlayer Player[2];  // 플레이어[0] 은 내가 되는것이고 플레이어 [1] 은 상대방이 되는것 기준은 플레이어 
   int Num_1,Num_2;

   Help SHelp;
   Exit SExit;
   
   
   gotoxy(40,15);
   cout << " 게임을 시작하려면 1을 눌러주세요." ;
   gotoxy(40,18);
   cout << " 도움말을 보려면 2를 눌러주세요. ";
   gotoxy(40,21);
   cout << " 게임을 종료하려면 3을 눌러주세요. ";

   int LifeBuffer=0;
   a=getch();
   char YNnum;
   system("cls");
  switch(a)
  {
   case '1':
      while(1)
      {
            
      for(int i=0;i<4;i++)
      {
         Set.randomCard();
         Set.showCard();
            
         Player[0].copy(Set.getMTrump());
            Player[1].copy(Set.getYTrump());
            
            gotoXY(80,5);
            cout << "Player1 Life: " ;
            Player[0].ShowLife();
            
            gotoXY(80,6);
            cout << "Player2 Life: " ;
            Player[1].ShowLife();            
         
               
         gotoXY(60,14);
         cout << "배팅을 시작하겠습니다.";
         
         if(i == 0 || i == 2)
         {
            gotoXY(60,15);
            cout << "Player1은 배팅을 하십시오: "; 
            Num_1=Player[0].Betting();
            
            gotoXY(60,16);
            cout << "Player2는 승락을 하시겠습니까?(Y/N) : ";
            cin >> YNnum;
            
            if(YNnum == 'Y' || YNnum == 'y')
            {
               Num_2=Num_1;
               Player[1].GetLostlife(0,Num_1);
               
               LifeBuffer=Num_1+Num_2;
            }
            
            else if( YNnum == 'N' || YNnum == 'n')
            {
               Num_2=Num_1/2;
               Player[1].GetLostlife(0,Num_2);
               
               LifeBuffer=Num_1+Num_2;
            }
            
            system("cls");
            Set.AllCard();   
            
            
            if(Player[0].CalCard() > Player[1].CalCard())
            {
               gotoXY(48,12);
               cout << "플레이어1 카드점수: " << Player[0].CalCard(); 
               gotoXY(40,15);
               cout << "Player1 이 우승합니다! Life는 플레이어 Life로 들어갑니다.";
               getch();
               Player[0].GetLostlife(1,LifeBuffer);
            }
            
            else if(Player[0].CalCard() < Player[1].CalCard())
            {
               gotoXY(40,15);
               cout << "Player2 이 우승합니다! Life는 플레이어 Life로 들어갑니다.";
               getch();
               Player[1].GetLostlife(1,LifeBuffer);
            }
            
            system("cls");
            gotoXY(40,15);
            cout << "다음 게임을 시작하겠습니다.";
            Sleep(1500);
            system("cls");
         }
         
         
         
         
         
         if( i == 1 || i == 3)
         {
            gotoXY(60,15);
            cout << "Player2은 배팅을 하십시오: "; 
            Num_1=Player[1].Betting();
            
            gotoXY(60,16);
            cout << "Player2는 승락을 하시겠습니까?(Y/N) : ";
            cin >> YNnum;
            
            if(YNnum == 'Y' || YNnum == 'y')
            {
               Num_2=Num_1;
               Player[0].GetLostlife(0,Num_1);

               LifeBuffer=Num_1+Num_2;
            }
            
            else if( YNnum == 'N' || YNnum == 'n')
            {
               Num_2=Num_1/2;
               Player[0].GetLostlife(0,Num_2);
               
               LifeBuffer=Num_1+Num_2;
            }
            
            system("cls");
            Set.AllCard();
            Sleep(1500);

            
            if(Player[0].CalCard() > Player[1].CalCard())
            {
               gotoXY(40,15);
               cout << "Player1 이 우승합니다! Life는 플레이어 Life로 들어갑니다.";
               
               Player[0].GetLostlife(1,LifeBuffer);
               getch();
               
            }
            
            else if(Player[0].CalCard() < Player[1].CalCard())
            {
               gotoXY(40,15);
               cout << "Player2 이 우승합니다! Life는 플레이어 Life로 들어갑니다.";
               
               Player[1].GetLostlife(1,LifeBuffer);
               getch();
            }
            
            system("cls");
            gotoXY(40,15);
            cout << "다음 게임을 시작하겠습니다.";
            Sleep(1500);
            system("cls");
         }
      }
      break;

   case '2':
      SHelp.ShowHelp();
      break;

   case '3':
      SExit.TheExit();
      break;
 
         }
   }
}

