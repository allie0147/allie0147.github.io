---
title: "Android Project: Kiwi Market"
date: 2020-09-28 23:31:00 -0400
categories: projects android
---

## Kiwi Market
#### 2020/08/21 ~ 2020/09/27
#### 중고거래 어플리케이션 "당근마켓" 클론코딩.
#### 위치(동네) 기반 중고 거래 서비스 및 1:1 채팅 시스템 구현.
***
1. 개발 환경   
**Language** *Java 1.8*    
**Android Sdk Version** *minSdkVersion 16, compileSdkVersion 30*    
**Server** *Firebase*    
**Database** *Firebase Realtime Database, Firebase Storage*    
**API** *Google Maps for Android*    
**Build** *Gradle*    
**Library**    
*Ted Permission* (<https://github.com/ParkSangGwon/TedPermission>)    
*Glide* (<https://github.com/bumptech/glide>)    
*android-ago* (<https://github.com/curioustechizen/android-ago>)    
*PageIndicatiorView* (<https://github.com/romandanylyk/PageIndicatorView>)    

2. 화면 구성   
*Splash*    
*Sign-up*    
*Login*    
*Forget Email*    
*Phone Auth*    
*Verify Address*    
*Main view*    
*Signle view for one post*    
*Posting format*        
*List of chat rooms*       
*A chat room*    
*Settings*    

3. 기능 기술     
- 1. *Splash*    
   - When server is checking, it supports to show error message and shut down application using **`Firebase Remote Config`**
   - If authenticated user starts app, go directly to main activity with user's location bundle
   - If not, go to login activity
- 2. *Sign-up*
  - Form validation: Email, Password, Nickname
  - (optional) User profile image: get permission **`READ_EXTERNAL_STORAGE and WRITE_EXTERNAL_STORAGE`**
  - Success: save profile and **`send verification email`** to user
- 3. *Login*
  - **`Firebase Authentication`**
  - Success
  - If new user logs in, go to verify phone number & location
  - If not, go main activity
  - Failure: show toast login failure text 
  - Email login
  - user **`MUST VERIFY`** their **`EMAIL`** before login
  - Google login
- 4. *Forgot email*
  - **`Firebase Authentication`**
  - receive user's phone number to verify user and Dialog shows user email
- 5. *Phone Auth*
  - **`Firebase Authentication`**
  - receive user's phone number and send **`verification code`**
  - user should enter a verification code which has **`6 digits`**
  - Success: save phone number and go next step to check user's location
- 6. *Verify Address*
  - Used **`Google Maps for Android API`**
  - Chcek user's current locaiton: get permission **`ACCESS_FINE_LOCATION and ACCESS_FINE_LOCATION`**
  - **`Address search`** service is supported 
  - Save address to **`Firebase Realtime DB`**
- 7. *Main*
  - **`Bottom navigation with 3 fragments and 1 activity`** : home, chat, settings and post(No.9)
  - **`CollapsingToolbar`** with user's current location
  - Show posts which are **`relevant to user's location`**
  - Recycler view with **`SwipeRefreshLayout`** and **`ShimmerFrameLayout`** ([link](https://facebook.github.io/shimmer-android/>))
  - Recycler view shows one image, title, detailed location, price and posted date
  - **`HomeFragment`** extends **`abstract class PostsFragment`** due to manage **`HomeFragment`** and **`SearchResultFragment`** (not yet released)
  - Use **`FirebaseRecyclerAdapter`** to make simple adapter
  - Get data from **`Realtime DB('posts'-'locationKey')`**
  - Clicking single post passes **`post key`** and **`location key`** to **`PostDetailActivity`**
- 8. *Signle view for one post*
  - **`NestedScollView`** has Toolbar, ViewPager, PageIndicator, fixed footer and detailed post
  - **`Toolbar`** has 2 menu items: nav_share, nav_delete
    - **`nav_share`**: sharing the post using **`Intent.ACTION_SEND`**(not yet released)
    - **`nav_delete`**: only visible for post author who can delete the post
    - **`ViewPager and PageIndicator`**
  - Use a **`Scrim`** for clear, readable icon
  - Viewpager set adapter and [pageIndicator](https://github.com/romandanylyk/PageIndicatorView) set viewpager **`.setAdapter(adapter)`**,            **`.addOnPageChangeListener`**, **`.setViewPager(binding.detailViewPager)`**
  - **`Detailed Post`**: **`ValueEventListener`** to get value from **`Firebase Realtime DB`**
  - **`Footer`**: **`Button`** which can make **`chatroom`** and go directly chatroom
  - Get single post data from **`Realtime DB('posts'-'locationKey'-'postKey')`** and **`Storage('postImages'-'locationKey'-'postKey')`**
- 9. *Posting format*
  - **`Toolbar`** with cancel, submit buttons
  - Uploading photos limit to five
  - User **`MUST`** upload at least **`ONE PHOTO`**, write **`TITLE, PRICE, DESCRIPTION`** and apply **`CATEGORY`**
  - If format is satisfied, form is submitted to **`Realtime DB('posts'-'locationKey'-'postKey(.push())')`** and **`Storage('postImages'-'locationKey'-                     'postKey(.push())`**
  - If not, Dialog will notify which one is not satisfied
- 10. *List of chat rooms*
  - **`RecyclerView`** with **`ChatListAdapter(FirebaseRecyclerAdapter)`**
  - Get data for list of chat rooms from **`Realtime DB('chat-lists'-'mUser.uId')`**
  - Adapter gets user's lists, the others' profile image and **`the most recent message`** for every single lists
``` java
public class ChatListAdapter extends FirebaseRecyclerAdapter<HashMap<String, String>, ChatListAdapter.ChatListViewHolder> {
   
    ...<codes>...

    protected void onBindViewHolder(@NonNull ChatListViewHolder holder, int position, @NonNull HashMap<String, String> model) {
    FirebaseDatabase.getInstance().getReference().child("user-chats").child(mUid)
                    .child(model.get("uId")).getRef().limitToLast(1)
                    .addListenerForSingleValueEvent(new ValueEventListener() {
                        @Override
                        public void onDataChange(@NonNull DataSnapshot snapshot) {
                            if (snapshot.getValue() != null) {
                                getValue(snapshot, holder, model);
                            } else {
                                FirebaseDatabase.getInstance().getReference().child("user-chats").child(model.get("uId"))
                                        .child(mUid).getRef().limitToLast(1)
                                        .addListenerForSingleValueEvent(new ValueEventListener() {
                                            @Override
                                            public void onDataChange(@NonNull DataSnapshot snapshot) {
                                                getValue(snapshot, holder, model);
                                            }
                                            @Override
                                            public void onCancelled(@NonNull DatabaseError error) {}
                                        });
                            }
                        }
                        @Override
                        public void onCancelled(@NonNull DatabaseError error) {}
                    });
    }
}
```
  - Clicking single list passes **`chatroom's user id`** to **`ChattingActivity`**
- 11. *Chat room*
  - One to one real-time chat service
  - **`Toolbar`** shows a nickname of the other user
  - Sending messages and image(not yet released)
  - Dependent on **`ItemViewType`**, **`Adapter`** creates **`ViewHolder`**
  - This is because chat bubbles with messages should attach to different position of a chat room (left and right)
``` java
public class ChatAdapter extends FirebaseRecyclerAdapter<Chat, ChatViewHolder> {
    private FirebaseUser mUser;
    private Chat mChat;
    private String sender = "";

    ...<codes>...

    @Override
    public int getItemViewType(int position) {
        mUser = FirebaseAuth.getInstance().getCurrentUser();
        mChat = getItem(position);
        if (mChat.getSender().equals(mUser.getUid())) {
            sender = mUser.getUid();
            return MESSAGE_TYPE_RIGHT;
        } else {
            sender = mChat.getSender();
            return MESSAGE_TYPE_LEFT;
        }
    }

    @NonNull
    @Override
    public ChatViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        LayoutInflater inflater = LayoutInflater.from(parent.getContext());
        if (viewType == MESSAGE_TYPE_RIGHT) { // current user's message
            return new ChatViewHolder(inflater.inflate(R.layout.chat_item_right, parent, false), MESSAGE_TYPE_RIGHT);
        } else { // the other's message
            return new ChatViewHolder(inflater.inflate(R.layout.chat_item_left, parent, false), MESSAGE_TYPE_LEFT);
        }
    }
}
```
  - The other's chat bubble has user profile
  - Each chat bubble has timestamp
- 12. *Settings*
  - Show user's profile
  - **`프로필 수정하기 Button`** send user to **`EditProfileActivity`**
    - Editting profile image and nickname is supported
  - User can reset address by **`동네 인증하기`**
  - **`Toolbar Menu Item`** for user log-out
