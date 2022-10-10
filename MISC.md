### Find Ids from Tweet Coding Question
```c++
#include <iostream>
#include <sstream>
#include <string>
#include <vector>
#include <unordered_set>
#include <unordered_map>
#include <algorithm>

using namespace std;

struct Id {
    string id;
    int mentions;
};

class Solver {
private:
    unordered_map<string, Id> m;
    
public:
    Solver(vector<string>& ids) {
        for (auto& id: ids) {
            if (m.find(id) == m.end()) {
                m[id] = {id.substr(1), 0};
            }
        }
    }
    
    void split(vector<string>& tweets) {
        for (auto& t: tweets) {
            unordered_set<string> currSeen;
            istringstream s(t);
            string currTweet;
            while(getline(s, currTweet, ' ')) {
                istringstream i(currTweet);
                string word;
                while (getline(i, word, ',')) {
                    if (m.find(word) == m.end() || currSeen.find(word) != currSeen.end()) 
                        continue;
                    currSeen.insert(word);
                    m[word].mentions++;
                }
            }
        }
    }
    
    void getResult() {
        vector<Id> ids;
        for (auto& pair: m) {
            ids.push_back(pair.second);
        }
        
        sort(ids.begin(), ids.end(), [&](Id id1, Id id2) {
            if (id1.mentions == id2.mentions)
                return id2.id < id1.id;
            return id2.mentions < id1.mentions;
        });
        
        for (auto& id: ids) {
            cout << id.id << "=" << id.mentions << endl;
        }
    }
};

int main() {
    // Write C++ code here
    vector<string> ids = {"@id51", "@id41", "@id41", "@id61", "@id92"};
    Solver s(ids);
    vector<string> tweets = {"@id51,@id51,@id61,@id92 hi!", "I'm hanging with @id41,@id2,@id51, today!", "@id92 what does @id14 @id61 plan on doing today?"}; 

    s.split(tweets);
    s.getResult();
    
    return 0;
}
```
