# Redux-saga 연동하기

### immer
- `npm i immer` 명령어를 사용하여 설치한다.
- 불변성을 지켜주는 라이브러이이다.

**/reducers/post.js** 
```javascript
import produce from 'immer';

...
const reducer = (state = initialState, action) => {
  // state가 draft 라는 이름으로 바뀐다. (불변성과 상관없이 변경 가능)
  // immer가 state를 불변성에 있게 새로운 상태로 생성한다.
  return produce(state, (draft) => {
    switch (action.type) {
      case ADD_POST_REQUEST:
        draft.addPostLoading = true;
        draft.addPostDone = false;
        draft.addPostError = null;
        break;
      case ADD_POST_SUCCESS:
        draft.addPostLoading = false;
        draft.addPostDone = true;
        draft.mainPosts.unshift(dummyPost(action.data));
        break;
      ...
      case ADD_COMMENT_SUCCESS: {
        const post = draft.mainPosts.find((v) => v.id === action.data.postId);
        post.Comments.unshift(dummyComment(action.data.content));
        draft.addCommentLoading = false;
        draft.addCommentDone = true;
        break;
      }
    ...
  });
}
```

```javascript
...

const reducer = (state = initialState, action) => {
  switch (action.type) {
    case ADD_POST_REQUEST:
      return (
        ...state,
        addPostLoading: true;
        addPostDone: false;
        addPostError: null,
      );
    ...
    case ADD_COMMENT_SUCCESS: {
      const postIndex = state.mainPosts.findIndex((v) => v.id === action.data.postId);
      const post = { ...state.mainPosts[postIndex] };
      post.Comments = [dummyComment(action.data.content), ...post.Comments];
      const mainPosts = [...state.mainPosts];
      mainPosts[postIndex] = post;
      return {
        ...state,
        mainPosts,
        addCommentLoading: false,
        addCommentDone: true,
      };
    }
  ...
}
```

### faker
- `npm i faker` 명령어로 설치한다.
- 더미데이터를 가져와서 사용할 수 있게 해준다.

**reducers/post.js**
```javascript
...
initialState.mainPosts = initialState.mainPosts.concat(
  Array(20).fill().map((v, i) => {
    id: shordId.generate(),
    User: {
      id: shortId.generate(),
      nickname: faker.name.findName();
    },
    content: faker.lorem.paragraph(),
    Images: [{
      src: faker.image.imageUrl(),
    }],
    Comments: [{
      User: {
        id: shortId.generate(),
        nickname: faker.name.findName();
      },
      content: faker.lorem.sentence(),
    }],
  })),
);
...
```
- placeholder: 이미지는 없지만 해당 영역을 채울 수 있는 라이브러리이다.
  
