#include<iostream>
#include<string>
#include<vector>
#include<fstream>
using namespace std;

class Question{
    private:
    string text;
    vector<string>options;
    int correctoption;
    public:
    Question(string t,vector<string> opts,int correct);
    void display() const;
    bool checkanswer(int choice) const;
    string getquestiontext() const;
};

class Exam{
    private:
    string subjectname;
    vector<Question>questions;
    public:
    Exam(string name);
    string getsubjectname() const;
    void addquestion(const Question& q);
    void removequestion(int index);
    void displayquestionsforteacher() const;
    vector<Question>& getquestion();
};

class Result{
    public:
    string studentname;
    string subject;
    int score;
    int remaininghealth;
    bool passed;
    Result(string name,string sub,int s,int h,bool p); 
};

class User{
    protected:
    string username;
    string password;
    public:
    User(string u,string p);
    string getusername() const;
    string getpassword() const;
    virtual void displaymenu()=0;
};

class Student:public User{
    private:
    int score;
    public:
    Student(string u,string p);
    void takeexam(Exam& exam);
    void viewmyresult();
    void displaymenu() override;
};

class Teacher:public User{
    public:
    Teacher(string u,string p);
    void managequestions();
    void mannualgrading();
    void displaymenu() override;
};

vector<Exam>examdatabase;
vector<Result>resultdatabase;

Question::Question(string t,vector<string> opts,int correct){
    text=t;
    options=opts;
    correctoption=correct;
}
void Question::display() const{
    cout<<"\n Question : "<<text<<endl;
    for(size_t i=0;i<options.size();i++){
        cout<<i+1<<". "<<options[i]<<endl;
    }
}
bool Question::checkanswer(int choice) const{
    return choice==correctoption;
}
string Question::getquestiontext() const{
    return text;
}

Exam::Exam(string name){
    subjectname=name;
}
string Exam::getsubjectname() const{
    return subjectname;
}
void Exam::addquestion(const Question& q){
    questions.push_back(q);
}
void Exam::removequestion(int index){
    if(index>=0&&index<questions.size()){
        questions.erase(questions.begin()+index);
        cout<<"\n Question remove successfully \n";
    }
    else{
        cout<<"invalid index \n";
    }
}
void Exam::displayquestionsforteacher() const{
    if(questions.empty()){
        cout<<"No questions yet \n";
        return;
    }
    else{
        for(size_t i=0;i<questions.size();i++){
            cout<<"[ "<<i+1<<"]  ";
            questions[i].display();
        }
    }
}
vector<Question>& Exam::getquestion(){
    return questions;
}

Result::Result(string name,string sub,int s,int h,bool p):studentname(name),subject(sub),score(s),remaininghealth(h),passed(p){}

User::User(string u,string p):username(u),password(p){}
string User::getusername() const{
    return username;
}
string User::getpassword() const{
    return password;
}

Student::Student(string u,string p):User(u,p),score(0){}
void Student::takeexam(Exam& exam){
    cout<<"\n========== EXAM START ==========\n";
    cout<<"Subject : "<<exam.getsubjectname()<<endl;
    vector<Question>& question=exam.getquestion();
    if(question.empty()){
        cout<<"No questions available.\n";
        return;
    }
    for(size_t i=0;i<question.size();i++){
        cout<<"\n---------------------------------\n";
        cout<<"Question "<<i+1<<endl;
        question[i].display();
        int answer;
        cout<<"Enter Answer (1-4) : ";
        cin>>answer;
        ofstream fout("attempts.txt",ios::app);
        fout<<username<<endl;
        fout<<exam.getsubjectname()<<endl;
        fout<<question[i].getquestiontext()<<endl;
        fout<<answer<<endl;
        fout<<"-----"<<endl;
        fout.close();
    }
    cout<<"\nExam submitted successfully.\n";
    cout<<"Teacher will evaluate your attempt later.\n";
}
void Student::viewmyresult(){
    cout<<"\n==MY Result==\n";
    bool found=false;
    for(const auto& res:resultdatabase){
        if(res.studentname==username){
            cout<<"Subject : "<<res.subject<<" | Score : "<<res.score<<" | Status : "<<(res.passed?"pass":"fail")<<endl; 
        }
    }
    if(!found){
        cout<<"No Exam record found\n";
    }
}
void Student::displaymenu(){
    int choice;
    do{
        cout<<"\n===Student Portal==\n1.Take Exam\n2.View my result\n3.Logout \nSelect : ";
        cin>>choice;
        if(choice==1){
            for(size_t i=0;i<examdatabase.size();i++){
                cout<<i+1<<"."<<examdatabase[i].getsubjectname()<<endl;
            }
            int subchoice;
            cin>>subchoice;
            if(subchoice>0&&subchoice<=examdatabase.size()){
                takeexam(examdatabase[subchoice-1]);
            }
        }
        else if(choice==2){
            viewmyresult();
        }
    }while(choice!=3);
}

Teacher::Teacher(string u,string p):User(u,p){}
void savequestiontofile(string subject,string qtext,string op1,string op2,string op3,string op4,int correct){
    ofstream fout("addedquestions.txt",ios::app);
    fout<<subject<<endl;
    fout<<qtext<<endl;
    fout<<op1<<endl;
    fout<<op2<<endl;
    fout<<op3<<endl;
    fout<<op4<<endl;
    fout<<correct<<endl;
    fout<<"-----"<<endl;
    fout.close();
}
void Teacher::managequestions(){
    cout<<"1. Object Oriented Programing\n";
    cout<<"2. Digital Logic Design\n";
    cout<<"3. Statistics and Probability\n";
    cout<<"4. Technical and Bussiness Writing\n";
    cout<<"5. Applied Physics\n";
    cout<<"6. Pakistan Studeies\n";
    cout<<"\nSelect Subject : (1-"<<examdatabase.size()<<") : ";
    int subchoice;
    cin>>subchoice;
    if(subchoice<=0||subchoice>examdatabase.size()){
        return;
    }
    Exam& selectedexam=examdatabase[subchoice-1];
    int action;
    cout<<"1.Add Question\n2.Remove Question\nSelect : ";
    cin>>action;
    if(action==1){
        string qtext,op1,op2,op3,op4;
        int correct;
        cout<<"Question Text : ";
        cin.ignore();getline(cin,qtext);
        cout<<"option 1 : ";
        cin.ignore();getline(cin,op1);
        cout<<"option 2 : ";
        cin.ignore();getline(cin,op2);
        cout<<"option 3 : ";
        cin.ignore();getline(cin,op3);
        cout<<"option 4 : ";
        cin.ignore();getline(cin,op4);
        cout<<"correct option(1-4) ";
        cin>>correct;
        selectedexam.addquestion(Question(qtext,{op1,op2,op3,op4},correct));
        savequestiontofile(selectedexam.getsubjectname(),qtext,op1,op2,op3,op4,correct);
        cout<<"Question added Successfully ";
    }
    else if(action==2){
        cout<<"Question number to remove \n";
        int qnum;
        cin>>qnum;
        selectedexam.removequestion(qnum-1);
    }
}
void Teacher::mannualgrading(){
    ifstream fin("attempts.txt");
    ofstream fout("temp.txt");
    if(!fin){
        cout<<"No attempts found\n";
        return;
    }
    string student;
    string subject;
    string question;
    string separator;
    int answer;
    while(true){
        if(!getline(fin,student))break;
        if(!getline(fin,subject))break;
        if(!getline(fin,question))break;
        if(!(fin>>answer))break;
        fin.ignore();
        if(!getline(fin,separator))break;
        cout<<"\n========================\n";
        cout<<"Student : "<<student<<endl;
        cout<<"Subject : "<<subject<<endl;
        cout<<"Question : "<<question<<endl;
        cout<<"Student Answer : "<<answer<<endl;
        int marks;
        cout<<"Give Marks : ";
        cin>>marks;
        bool passed=(marks>0);
        resultdatabase.push_back(Result(student,subject,marks,0,passed));
    }
    fin.close();
    fout.close();
    remove("attempts.txt");
    rename("temp.txt","attempts.txt");
}
void Teacher::displaymenu(){
    int choice;
    do{
        cout<<"\n===Teacher Portal==\n1.Manage Questions\n2.Mannual Grading\n3.Logout\nSelect :\n";
        cin>>choice;
        if(cin.fail()){
            cin.clear();
            cin.ignore(10000,'\n');
            cout<<"Invalid input. Please enter a valid number.\n";
            continue;
        }
        if(choice==1){
            managequestions();
        }
        else if(choice==2){
            mannualgrading();
        }
    }while(choice!=3);
}

void loadquestionsfromfile(){
    ifstream fin("addedquestions.txt");
    if(!fin){
        return;
    }
    string subject;
    string qtext;
    string op1,op2,op3,op4;
    string separator;
    int correct;
    while(getline(fin,subject)){
        getline(fin,qtext);
        getline(fin,op1);
        getline(fin,op2);
        getline(fin,op3);
        getline(fin,op4);
        fin>>correct;
        fin.ignore();
        getline(fin,separator);
        for(size_t i=0;i<examdatabase.size();i++){
            if(examdatabase[i].getsubjectname()==subject){
                examdatabase[i].addquestion(Question(qtext,{op1,op2,op3,op4},correct));
            }
        }
    }
    fin.close();
}
void populateinitialdata(){
    examdatabase.push_back(Exam("Object Oriented Programing"));
    examdatabase.push_back(Exam("Digital Logic Design"));
    examdatabase.push_back(Exam("Statistics and Probability"));
    examdatabase.push_back(Exam("Technical and Bussiness Writing"));
    examdatabase.push_back(Exam("Applied Physics"));
    examdatabase.push_back(Exam("Pakistan Studeies"));
    examdatabase[0].addquestion(Question("If a new AI 'NeuralNet' class derives features from a base 'Model' class, what OOP concept is this?",{"Encapsulation","Polymorphism","Inheritance","Abstraction"},3));
    examdatabase[1].addquestion(Question("Which 7400 series IC is commonly known as a Quad 2-Input AND gate?",{"7404","7408","7432","7400"},2));
    examdatabase[2].addquestion(Question("As a sample size grows, its mean gets closer to the average of the whole population. This rule is called the...",{"Central Limit Theorem","Law of Large Numbers","Standard Deviation","P-Value"},2));
    examdatabase[3].addquestion(Question("Which communication skill is most important in presentations?",{"Sleeping","Clear speaking","Ignoring audience","Reading silently"},2));
    examdatabase[4].addquestion(Question("The SI unit of force",{"joule","watt","newton","pascal"},3));
    examdatabase[5].addquestion(Question("Who is known as the founder of Pakistan?",{"Quaid e Azam","Allama Iqbal","Sir syed Ahmed Khan","Liaqat Ali Khan"},1));
}
int main(){
    populateinitialdata();
    loadquestionsfromfile();
    Student s1("kashif","123");
    Teacher t1("faheem","admin");
    int mainchoice;
    cout<<"=========================================\n";
    cout<<"              EXAM SYSTEM                \n";
    cout<<"=========================================\n";
    do{
        cout<<"\n1.Student login\n2.Teacher Login\n3.Exit\nSelect : \n";
        cin>>mainchoice;
        if(mainchoice==1){
            string u,p;
            cout<<"User : ";
            cin>>u;
            cout<<"Password : ";
            cin>>p;
            if(u==s1.getusername()&&p==s1.getpassword()){
                s1.displaymenu();
            }
            else{
                cout<<"invalid credentials \n";
            }
        }
        else if(mainchoice==2){
            string u,p;
            cout<<"User : ";
            cin>>u;
            cout<<"Password : ";
            cin>>p;
            if(u==t1.getusername()&&p==t1.getpassword()){
                t1.displaymenu();
            }
            else{
                cout<<"invalid credentials \n";
            }
        }
    }while(mainchoice!=3);
    return 0;
}
