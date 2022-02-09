# Next-Auth-With-Kakao&mongoDB

**목표**
리엑트에서 카카오톡 로그인 인증과 사용자 정보를 **mongoDB**에 기록한다. 

####  Step.1 New Project
**[Next.js](https://nextjs.org/) 프레임워크 사용.**
```
yarn create next-app next-auth-with-kakao
yarn  add next-auth @next-auth/mongodb-adapter mongodb
```

#### Step.2 Add API route

page/api/auth/[...nextauth].js
```
import NextAuth from  "next-auth";
import KakaoProvider from  "next-auth/providers/kakao";

export default NextAuth({
	provider:[
		KakaoProvider({
			clientId: process.env.KAKAO_CLIENT_ID,
			clientSecret: process.env.KAKAO_CLIENT_SECRET
		}),
	],
});
```
> **clientId:** kakao에서 발급한 secret key
>  **clientSecret:** 의미없는 문자열

#### Step.3 Connect MongoDB

lib/mongodb.js
```
import { MongoClient } from "mongodb"

const uri = process.env.MONGODB_URI;
const options = {
    useUnifiedTopology: true,
    useNewUrlParser: true,
}

let client;
let clientPromise;

if (!process.env.MONGODB_URI) {
    throw new Error("Please add your Mongo URI to .env.local")
};

if (process.env.NODE_ENV === "development") {
    if (!global._mongoClientPromise) {
        client = new MongoClient(uri, options)
        global._mongoClientPromise = client.connect()
    };
    clientPromise = global._mongoClientPromise
} else {
    client = new MongoClient(uri, options)
    clientPromise = client.connect()
};

export default clientPromise;
```

page/api/auth/[...nextauth].js
```
import NextAuth from  "next-auth";
import KakaoProvider from  "next-auth/providers/kakao";
import clientPromise from "../../../util/mongodb";
import { MongoDBAdapter } from "@next-auth/mongodb-adapter";

export default NextAuth({
	...
    adapter: MongoDBAdapter(clientPromise),
});
```
> mongodb-adapter가 mongodb에 정보를 자동으로 저장해 준다.
![enter image description here](https://next-auth.js.org/assets/images/nextauth_v4_schema-9d6746cfdef30cb1a4c573edb0cc8070.png)

##
.env.local
```
MONGODB_URI= 
KAKAO_CLIENT_ID=
KAKAO_CLIENT_SECRET=
NEXTAUTH_SECRET=
NEXTAUTH_URL= http://localhost:PORTNUMBER
```