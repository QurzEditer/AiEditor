# 批注（或评论）

批注功能和 Word 的批注评论功能类似，可以选择一段文字对齐进行批注，如下图所示：

![](../../assets/image/comment1.png)

> PS：此功能在 Pro 版本才有，开源版没有这个功能。 Pro 版预览地址：http://pro.aieditor.com.cn

## 使用方法

```typescript
new AiEditorPro({
    element: "#aiEditor",
    comment: {
        enable: true,
        floatable: false,
        containerClassName: "comment-container",

        // currentAccountAvatar: "https://aieditor.com.cn/logo.png",
        currentAccount:"Miachel Yang",
        currentAccountId: "1",
        enableWithEditDisable: true,

        // commentCreateDisable: true,
        // commentUpdateDisable: true,
        // commentDeleteDisable: true,
        // commentReplyDisable: true,

        queryAllComments: () => {
            // 查询当前文档的所有评论，返回 CommentInfo[] 或者 Promise<CommentInfo[]>
        },

        queryCommentsByIds: (commentIds: string[]) => {
            // 查询当前文档的所有评论，返回 CommentInfo[] 或者 Promise<CommentInfo[]>
        },

        queryMyComments: () => {
            // 查询 “我的评论”，返回 CommentInfo[] 或者 Promise<CommentInfo[]>
        },

        queryCommentsByIds: (commentIds) => {
            // 根据多个文档id，查询多条评论，返回 CommentInfo[] 或者 Promise<CommentInfo[]>
        },

        onCommentActivated: (commentId?: string[] | null) => {
            // 当评论区域获得焦点
        },
        
        onCommentCreate: (comment) => {
            // 当评论被创建时，，返回 CommentInfo 或者 Promise<CommentInfo>
        },

        onCommentDeleteConfirm:(comment)=>{
            // 当评论被删除时，弹出确认框，返回 boolean 或者  Promise<boolean>;
        },

        onCommentDelete: (commentId) => {
            // 当评论被删除时， 返回 boolean 或者  Promise<boolean>;
        },

        onCommentUpdate: (commentId, content) => {
            // 当评论被修改时，返回 boolean 或者  Promise<boolean>;
        },
    },
})
```

- **enable**: 是否启用评论功能
- **floatable**: 评论的内容，是否是跟评论区域位置浮动的，默认为 true
- **containerClassName**: 自定义评论区域的 class 名称，用于自定义样式
- **currentAccountAvatar**: 在 `floatable = false` 时，配置显示头像
- **currentAccount**: 配置当前用户的名称，用于显示在评论中
- **currentAccountId**: 配置当前用户的id
- **enableWithEditDisable**: 是否在只读模式下，也开启批注（评论）的功能，默认为 false
- **commentCreateDisable**: 是否关闭 **批注** 功能，默认为 false
- **commentUpdateDisable**: 是否关闭 **批注修改** 功能，默认为 false
- **commentDeleteDisable**: 是否关闭 **批注删除** 功能，默认为 false
- **commentReplyDisable**: 是否关闭 **批注回复** 功能，默认为 false
- **`onCommentActivated: (commentId?: string[] | null) => void`**: 评论获得焦点时，会触发此回调
- **`onCommentCreate: (comment: CommentInfo) => CommentInfo | Promise<CommentInfo>`**:  监听评论被创建，此时我们应该把评论内容保存到数据库，并返回完整的评论信息
- **`onCommentDeleteConfirm:(commentInfo: CommentInfo) => boolean | Promise<boolean>;`**:  监听评论被删除，弹出确认框，返回 boolean 或者  `Promise<boolean>`
- **`onCommentDelete:(commentId: string) => boolean | Promise<boolean>`**:  监听评论被删除，此时应该同步删除数据库的评论
- **`onCommentUpdate:(commentId: string, content: string) => boolean | Promise<boolean>`**:  监听评论被修改
- **`queryAllComments:(commentIds: string[]) => CommentInfo[] | Promise<CommentInfo[]>`**:  查询所有的评论（当配置 `floatable: false` 是有效）
- **`queryMyComments: () => CommentInfo[] | Promise<CommentInfo[]>`**:  查询 我的评论（当配置 `floatable: false` 是有效）
- **`queryCommentsByIds:() => CommentInfo[] | Promise<CommentInfo[]>`**:  根据多条评论 id，查询所有的评论（当配置 `floatable: true` 是有效）


需要保存的评论（批注）信息 `CommentInfo` 代码内容如下：

```ts
export interface CommentInfo {
    id?: string,
    pid?: string,
    content?: string,
    account?: string,
    accountId?: string,
    avatar?: string,
    createdAt?: string,
    mainColor?: string,
    commentFor?: string,
    replyAccount?: string,
    replyAccountId?: string,
    children?: CommentInfo[],
}
```

字段含义如下：
- **id**: 评论的 id，如果为空，则表示新增的评论
- **pid**: 父级评论的 id，如果为空，则表示根级评论
- **content**: 评论的内容
- **account**: 评论的用户名称
- **accountId**: 评论的用户id
- **avatar**: 评论的用户头像
- **createdAt**: 创建时间
- **mainColor**: 评论的背景色
- **commentFor**: 被评论的内容（也就是编辑器中选中的文字）
- **replyAccount**: 评论的回复用户名称
- **replyAccountId**: 评论的回复用户id
- **children**: 子级评论

当发生评论时，会触发 `onCommentCreate` 回调，编辑器给出的 comment 信息如下：

```ts
account:"Miachel Yang"
accountId:"1"
commentFor:"AiEditor is a next-generation "
content:"不错"
id:"45bde8a5-1416-4c46-9b7e-73cb4d5bff2d"
```

开发者需要在 `onCommentCreate` 回调中，补充完整信息，并返回完整的评论信息。

例如：
```ts
onCommentCreate: (comment) => {
    // 补充完整信息
    comment = {
        ...comment,
        account: "杨福海",
        mainColor: colors[Math.floor(Math.random() * colors.length)],
        createdAt: "05-26 10:23",
    } as CommentInfo;
```

此时应该把评论内容保存到数据库，并返回完整的评论信息


## 示例代码

以下的示例代码，是使用了 LocalStorage 来保存评论内容，方便开发者理解和测试，在实际使用中，请通过 http 请求保存评论内容。

```typescript
const colors = ['#3f3f3f', '#938953', '#548dd4', '#95b3d7',
    '#d99694', '#c3d69b', '#b2a2c7', '#92cddc', '#fac08f'];
new AiEditorPro({
    element: "#aiEditor",
    comment: {
        enable: true,
        // floatable: false,
        containerClassName: "comment-container",

        // currentAccountAvatar: "https://aieditor.com.cn/logo.png",
        currentAccount: "杨福海",
        currentAccountId: "1",
        enableWithEditDisable: true,
        // commentCreateDisable: true,
        // commentUpdateDisable: true,
        // commentDeleteDisable: true,
        // commentReplyDisable: true,
        onCommentActivated: (_commentIds) => {
            // console.log("onCommentActivated---->", commentId)
        },

        // 注意： queryAllComments 只有在设置 `floatable = false` 是配置才有意义
        queryAllComments: () => {
            const allCommentsString = localStorage.getItem("all-comments");
            const allCommentIds = allCommentsString ? JSON.parse(allCommentsString) : [];
            const allComments = [] as any[];
            allCommentIds.forEach((commentId: any) => {
                const contentJSON = localStorage.getItem(commentId);
                if (contentJSON) allComments.push(JSON.parse(contentJSON));
            })
            return allComments;
        },

        queryCommentsByIds: (commentIds) => {
            const allComments = [] as any[];
            if (commentIds) commentIds.forEach((commentId: any) => {
                const contentJSON = localStorage.getItem("comment-" + commentId);
                if (contentJSON) allComments.push(JSON.parse(contentJSON));
            })


            return new Promise((resolve) => {
                setTimeout(() => {
                    resolve(allComments);
                }, 200)
            });
        },


        onCommentCreate: (comment) => {
            console.log("onCommentCreate---->", comment)
            comment = {
                ...comment,
                account: "杨福海",
                // avatar: Math.floor(Math.random() * 10) > 3 ? "https://aieditor.dev/assets/image/logo.png" : undefined,
                mainColor: colors[Math.floor(Math.random() * colors.length)],
                createdAt: "05-26 10:23",
            } as CommentInfo;

            localStorage.setItem("comment-" + comment.id, JSON.stringify(comment));

            const allCommentsString = localStorage.getItem("all-comments");
            const allComments = allCommentsString ? JSON.parse(allCommentsString) : [];

            if (comment.pid) {
                const parentCommentJSON = localStorage.getItem("comment-" + comment.pid);
                if (parentCommentJSON) {
                    const parentComment = JSON.parse(parentCommentJSON);
                    if (parentComment.children) {
                        parentComment.children.unshift(comment)
                    } else {
                        parentComment.children = [comment]
                    }
                    localStorage.setItem("comment-" + comment.pid, JSON.stringify(parentComment));
                }
            } else {
                allComments.push("comment-" + comment.id)
            }


            localStorage.setItem("all-comments", JSON.stringify(allComments));

            return new Promise((resolve) => {
                setTimeout(() => {
                    resolve(comment);
                }, 200);
            });

            // return new Promise((resolve) => {
            //     resolve(comment)
            // })
        },


        onCommentDeleteConfirm: (comment) => {
            console.log("onCommentDeleteConfirm---->", comment)
            // return confirm("确认要删除吗");
            return new Promise((resolve) => {
                setTimeout(() => {
                    resolve(confirm("确认要删除吗"));
                }, 200)
            });
        },

        onCommentDelete: (commentId) => {
            console.log("onCommentDelete---->", commentId)

            const commentInfo = JSON.parse(localStorage.getItem("comment-" + commentId)!);
            if (commentInfo.pid) {
                const parentCommentJSON = localStorage.getItem("comment-" + commentInfo.pid);
                if (parentCommentJSON) {
                    const parentComment = JSON.parse(parentCommentJSON);
                    if (parentComment.children) {
                        parentComment.children = parentComment.children.filter((item: any) => item.id !== commentId)
                        localStorage.setItem("comment-" + commentInfo.pid, JSON.stringify(parentComment));
                    }
                }
            }
            localStorage.removeItem("comment-" + commentId);

            // return true;

            return new Promise((resolve) => {
                setTimeout(() => {
                    resolve(true);
                }, 200)
            });
        },

        onCommentUpdate: (commentId, content) => {
            // return false

            console.log("onCommentUpdate---->", commentId, content)
            const commentInfo = JSON.parse(localStorage.getItem("comment-" + commentId)!);
            commentInfo.content = content
            localStorage.setItem("comment-" + commentId, JSON.stringify(commentInfo))

            return new Promise((resolve) => {
                setTimeout(() => {
                    resolve(true);
                }, 200)
            });
        }
    },
})
```


以下是一个使用 HTTP 请求的示例：

```typescript
const postRequest = (url: string, data: any) => {
    const options = {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify({
            data
        }),
    };

    return fetch(url, options);
}

const getRequest = (url: string) => {
    return fetch(url, {method: 'GET'});
}


const colors = ['#3f3f3f', '#938953', '#548dd4', '#95b3d7',
    '#d99694', '#c3d69b', '#b2a2c7', '#92cddc', '#fac08f'];
new AiEditorPro({
    element: "#aiEditor",
    comment: {
        enable: true,
        // floatable: false,
        containerClassName: "comment-container",

        // currentAccountAvatar: "https://aieditor.com.cn/logo.png",
        currentAccount: "杨福海",
        currentAccountId: "1",
        enableWithEditDisable: true,
        // commentCreateDisable: true,
        // commentUpdateDisable: true,
        // commentDeleteDisable: true,
        // commentReplyDisable: true,
        onCommentActivated: (_commentIds) => {
            // console.log("onCommentActivated---->", commentId)
        },

        // 注意： queryAllComments 只有在设置 `floatable = false` 是配置才有意义
        queryAllComments: () => {
            return new Promise((resolve) => {
                // 根据某个文档 id 来获取该文档的所有评论
                getRequest("http://127.0.0.1:8080/api/comments?documentId=1")
                    .then(resp => resp.json())
                    .then(data => {
                        // 假设后台返回的 comments 数组放在 json 的 comments 中，例如 { comments: [{}, {}, {}] }
                        resolve(data.comments)
                    })
            })
        },

        queryCommentsByIds: (commentIds) => {
            return new Promise((resolve) => {
                getRequest("http://127.0.0.1:8080/api/comments?commentIds=" + commentIds)
                    .then(resp => resp.json())
                    .then(data => {
                        // 假设后台返回的 comments 数组放在 json 的 comments 中，例如 { comments: [{}, {}, {}] }
                        resolve(data.comments)
                    })
            });
        },


        onCommentCreate: (comment) => {
            console.log("onCommentCreate---->", comment)
            comment = {
                ...comment,
                account: "杨福海",
                avatar: "https://aieditor.dev/assets/image/logo.png",
                mainColor: colors[Math.floor(Math.random() * colors.length)],
                createdAt: "05-26 10:23",
            } as CommentInfo;


            return new Promise((resolve) => {
                postRequest("http://127.0.0.1:8080/api/commentCreate", comment)
                    .then(resp => resp.json())
                    .then(data => {
                        resolve(data.comment);
                        // 如果后端不返回 comment 对象，也可以直接
                        // resolve(comment); // 用上方填充内容的 comment
                    })
            });
        },


        onCommentDeleteConfirm: (comment) => {
            console.log("onCommentDeleteConfirm---->", comment)
            // return confirm("确认要删除吗");
            return new Promise((resolve) => {
                setTimeout(() => {
                    resolve(confirm("确认要删除吗"));
                }, 200)
            });
        },

        onCommentDelete: (commentId) => {
            return new Promise((resolve) => {
                postRequest("http://127.0.0.1:8080/api/commentDelete", {"id": commentId})
                    .then(resp => resolve(true))
            });
        },

        onCommentUpdate: (commentId, content) => {
            return new Promise((resolve) => {
                postRequest("http://127.0.0.1:8080/api/commentUpdate", {"id": commentId,"content": content})
                    .then(resp => resolve(true))
            });
        }
    },
})
```
