# rsschool-cv
void readBinFile(Platform &platform, string fileName) {
  ifstream fileRead;  
  Subscriber newSubs;
  BinSubscriber binSubs;

  fileRead.open(fileName, ios::in | ios::binary);
  
  if (fileRead.is_open()) {
    //erase old data from platfrom.subscribers
    platform.subscribers.clear();
  //read binary file
    fileRead.seekg(1*sizeof(BinPlatform),ios::beg); //jump over the platform struct
    while (fileRead.read((char *)&binSubs, sizeof(BinSubscriber))) {
      newSubs.id = binSubs.id;
      newSubs.name = binSubs.name;
      newSubs.email = binSubs.email;
      newSubs.mainIp = binSubs.mainIp;
      if (strlen(binSubs.mainIp) != 0) {
        newSubs.ips.push_back(binSubs.mainIp);
      } 
      platform.subscribers.push_back(newSubs);
      platform.nextId = newSubs.id + 1;
      newSubs.ips.clear(); 
    } fileRead.close();
    int x = platform.subscribers.size();
    platform.nextId = platform.subscribers[x-1].id + 1;
  } else {
    error(ERR_FILE);
    return;
  }
        
}

//makes sure user wants to erase all data and load new data from binary file
void loadData(Platform &platform) {
  char c;
  string fileName;
  cout << "All data will be erased. Continue? [y/n]: " << endl;
  cin >> c;
  cin.get();
  
  while (c != 'n' && c != 'N') {
    if (c == 'y' || c == 'Y') { 
        cout << "Enter filename:" << endl;
        getline(cin, fileName);     
        readBinFile(platform, fileName);
        break;
    } else {
      cout << "All data will be erased. Continue? [y/n]: " << endl;
      cin >> c;
      cin.get();
    }
  }
  return;
}

void saveData(const Platform &platform) {
  ofstream fileWrite;
  BinPlatform binPlat;
  string fileName;
  
  cout << "Enter filename:" << endl;
  getline(cin, fileName);

//first open binaryfile we are writing to and write binPlatform to it
  fileWrite.open(fileName, ios::out | ios::binary);
  if (fileWrite.is_open()) {
    strncpy(binPlat.name, platform.name.c_str(), KMAXSTRING - 1); 
    binPlat.name[KMAXSTRING-1] = '\0';
    binPlat.nextId = platform.nextId;
    fileWrite.write((const char *)&binPlat, sizeof(BinPlatform));
    //then write all subscribers one by one to the binary file
    for (unsigned int i = 0; i < platform.subscribers.size(); i++) { //loop through all the subscribers from vector
      BinSubscriber binSub;
      //copy name, email and mainIp to binStructure
      strncpy(binSub.name, platform.subscribers[i].name.c_str(), KMAXSTRING - 1); 
      binSub.name[KMAXSTRING-1] = '\0';
      strncpy(binSub.email, platform.subscribers[i].email.c_str(), KMAXSTRING - 1);
      binSub.email[KMAXSTRING-1] = '\0';
      strncpy(binSub.mainIp, platform.subscribers[i].mainIp.c_str(), KMAXIP - 1);
      binSub.mainIp[KMAXSTRING-1] = '\0';

      binSub.id = platform.subscribers[i].id;
      //write binSub to the binaryfile
      fileWrite.write((const char*)&binSub, sizeof(BinSubscriber));
    }
    fileWrite.close();
  } else {
    error(ERR_FILE);
    return;
  }

}

int main(int argc, char *argv[]) {  
  bool argumentsOk = false;
  Platform platform;
  platform.name = "Streamflix";
  platform.nextId = 1;

  if (argc == 3 || argc == 5) { //we know we can have 1, 3 or 5 arguments when argumentline is correct: prgram name, 1st argv, filename, 2nd argv, filename
    argumentsOk = argumentHandler(argc, argv, platform);
  } else if (argc == 1) { //this means, that there is no additional arguments given on command line, so argumentsOk = true and we go to main menu
    argumentsOk = true;
  } else { //for example 2, 4 or more than 5 arguments = wrong arguments, argumentsOk stays false and program just ends without showing main menu
    error(ERR_ARGS); 
  }
//if argumentsOk is true, show main menu and do things through it
  if (argumentsOk) {
    char option;
    do {
      showMainMenu();
      cin >> option;
      cin.get();

      switch (option) {
        case '1':
          showSubscribers(platform);
          break;
        case '2':
          addSubscriber(platform);
          break;
        case '3':
          addSubscriberIp(platform);
          break;
        case '4':
          deleteSubscriber(platform);
          break;
        case '5':
          importExportMenu(platform);
          break;
        case 'q':
          break;
        default:
          error(ERR_OPTION);
      }
    } while (option != 'q');

    return 0;
  }
}
