
### 시작 배경
  
> It is actually mostly like us two guys in a room riffing.  
  
2024년 7월경  
**David Soria Parra*** / Justin Spahr-Summers 둘이 아이디어를 주고받으면서 시작.  
  
>From my development tooling background, I quickly got frustrated by the idea that, you know, on one hand side, I have Cloud Desktop, which is this amazing tool with artifacts, which I really enjoyed. But it was very limited to exactly that feature set. And it was there was no way to extend it. And on the other hand side, I like work in IDEs, which could greatly like act on like the file system and a bunch of other things. But then they don't have artifacts or something like that. And so what I constantly did was just copy. Things back and forth on between Cloud Desktop and the IDE, and that quickly got me, honestly, just very frustrated.  
  
 Clause Desktop과 IDE 사이를 copy and past 하는게 너무 답답했음.  
  
>"And so it's very quickly that you see that this is clearly like an M times N problem. Like you have multiple like applications. And multiple integrations you want to build and like, what that is better there to fix this than using a protocol."  
  
 이 M x N 문제를 프로토콜로 해결해보면 어떨까?  
  
>[!Note]  
>N times N problem  
>**M개의 애플리케이션**과 **N개의 서비스/시스템**이 있을 때, 각각을 직접 연결하려면 **M × N개의 개별 통합**을 구축해야 하는 문제.  
>5개 애플리케이션 × 4개 외부 API = 20개의 서로 다른 통합 코드  
>각각 다른 인증, 데이터 형식, 에러 처리 방식  

---
### **표준 프로토콜 도입**  
  
```  
애플리케이션들 ← 단일 프로토콜 → 통합 레이어 ← 개별 연결 → 외부 시스템들  
```  
  
 **복잡도 감소**  
  
- M × N → M + N 개의 연결  
- 5 × 4 = 20개 → 5 + 4 = 9개 연결  
  
>"we definitely did take heavy inspiration from LSP. David had much more prior experience with it than I did working on developer tools... LSP largely, I think, solved this problem by creating this common language that they could all speak and that, you know, you can have some people focus on really robust language server implementations, and then the IDE developers can really focus on that side. And they both benefit."  
  
 LSP에서 많은 영감을 받음.  
  
>Yeah, I can chat about this a bit. I think fundamentally we every sort of primitive that we thought through, we thought from the perspective of the application developer first, like if I'm building an application, whether it is an IDE or, you know, call a desktop or some agent interface or whatever the case may be, what are the different things that I would want to receive from like an integration? And I think once you take that lens, it becomes quite clear that that tool calling is necessary, but very insufficient.  
  
저는 기본적으로, 우리가 고안한 모든 종류의 프리미티브를 애플리케이션 개발자의 관점에서 먼저 생각.  
  
IDE이든 Cloud Desktop이든 에이전트 인터페이스이든 어떤 애플리케이션을 만드는 상황에서도, 통합을 통해 개발자가 어떤 기능을 받고 싶어할지를 고민.  
  
그리고 그런 시각으로 보면, 도구 호출(tool calling)이 분명 필요하지만 그것만으로는 크게 부족하다는 사실이 명확해짐.  
  
>Like there are many other things you would want to do besides just get tools. And plug them into the model and you want to have some way of differentiating what those different things are  
  
단순히 도구만 모델에 연결하는 것 외에도 하고 싶은 일이 많기 때문인데, 서로 다른 기능을 구분할 수 있는 방법이 필요함.  

>[!important]
>주요 내용
>- Claude Desktop과 IDE 사이의 반복적인 복사-붙여넣기 작업의 불편함에서 시작
>- M×N 통합 문제를 해결하기 위한 표준 프로토콜의 필요성 인식
>- LSP(Language Server Protocol)의 성공 사례에서 영감을 받음
>- 애플리케이션 개발자 관점에서 설계 시작
>- 단순한 도구 호출(tool calling)을 넘어서는 더 풍부한 기능 제공 목표

---
### mcp 의 3 가지 primitive type
  
#### **1. Tools (도구)**  
  
> So the kind of core primitives that we started MCP with, we've since added a couple more, but the core ones are really tools, which we've already talked about. It's like adding, adding tools directly to the model or function calling is sometimes called  
  
MCP로 시작한 핵심 원시 타입들. 이후 몇 개 더 추가했지만 핵심은 도구들, 모델에 직접 도구를 추가하거나 함수 호출이라고도 불림.  
  
#### **2. Resources (리소스)**  
  
> resources, which is basically like bits of data or context that you might want to add to the context. So excuse me, to the, to the model context. And this, this is the first primitive where it's like, we, we. Decided this could be like application controlled, like maybe you want a model to automatically search through and, and find relevant resources and bring them into context. But maybe you also want that to be an explicit UI affordance in the application where the user can like, you know, pick through a dropdown or like a paperclip menu or whatever, and find specific things and tag them in. And then that becomes part of like their message to the LLM. Like those are both use cases for resources.  
  
기본적으로 모델 컨텍스트에 추가하고 싶은 데이터나 컨텍스트 조각들, 애플리케이션이 제어할 수 있다고 결정한 첫 번째 원시 타입.  
  
모델이 자동으로 관련 리소스를 검색해서 컨텍스트에 가져오거나, 사용자가 드롭다운이나 클립 메뉴 등의 명시적 UI를 통해 특정 항목을 선택하고 태그하여 LLM 메시지의 일부로 만드는 두 가지 사용 사례 모두 가능.  
  
#### **3. Prompts (프롬프트)**  
  
> And then the third one is prompts. Which are deliberately meant to be like user initiated or. Like. User substituted. Text or messages. So like the analogy here would be like, if you're an editor, like a slash command or something like that, or like an at, you know, auto completion type thing where it's like, I have this kind of macro effectively that I want to drop in and use.  
  
세 번째는 프롬프트. 의도적으로 사용자가 시작하거나 대체하는 텍스트나 메시지로 설계됨.  
  
에디터의 슬래시 명령어나 @ 자동완성 같은 것으로, 드롭인해서 사용할 수 있는 매크로 같은 개념.  
  
### **애플리케이션 개발자 관점의 차별화**  
  
> And we have sort of expressed opinions through MCP about the different ways that these things could manifest, but ultimately it is for application developers to decide, okay, you, you get these different concepts expressed differently. Um, and it's very useful as an application developer because you can decide. The appropriate experience for each, and actually this can be a point of differentiation to, like, we were also thinking, you know, from the application developer perspective, they, you know, application developers don't want to be commoditized. They don't want the application to end up the same as every other AI application.  
  
MCP를 통해 이런 것들이 어떻게 구현될 수 있는지 의견을 표현했지만, 궁극적으로는 애플리케이션 개발자가 각기 다른 개념들을 어떻게 표현할지 결정하는 것.  
  
각각에 적절한 경험을 결정할 수 있어서 차별화 포인트가 될 수 있음. 애플리케이션 개발자들은 상품화되길 원하지 않고, 다른 AI 애플리케이션들과 똑같아지길 원하지 않음.  
  
> So like, what are the unique things that they could do to like create the best user experience even while connecting up to this big open ecosystem of integration?  
  
거대한 오픈 통합 생태계에 연결하면서도 최고의 사용자 경험을 만들기 위해 할 수 있는 고유한 것들이 무엇인가?  

>[!important]
>주요 내용
>- **Tools**: 모델이 직접 호출하는 기능들 (모델 주도적)
>- **Resources**: 데이터/컨텍스트 조각들 (애플리케이션/사용자 제어 가능)
>- **Prompts**: 사용자가 시작하는 텍스트/명령어 매크로
>- 각 프리미티브는 애플리케이션 개발자가 자유롭게 구현 방식을 선택 가능
>- 애플리케이션의 차별화를 위한 유연성 제공

---  
###  **실제 구현 사례**  
  
**첫 번째 구현 - Prompts**  
  
> The, the very first implementation in that is actually a prompt implementation. It doesn't deal with tools. And, and it, we found this actually quite useful because what it allows you to do is, for example, build an MCP server that takes like a backtrack. So it's, it's not necessarily like a tool that literally just like rawizes from Sentry or any other like online platform that, that tracks your, your crashes. And just lets you pull this into the context window beforehand. And so it's quite nice that way that it's like a user driven interaction that you does the user decide when to pull this in and don't have to wait for the model to do it.  
  
첫 번째 구현은 실제로 프롬프트 구현이었고 도구를 다루지 않았음.  
  
예를 들어 백트레이스를 가져오는 MCP 서버를 구축할 수 있어서 매우 유용했음.  
  
Sentry나 다른 온라인 크래시 추적 플랫폼에서 단순히 원시 데이터를 가져오는 도구가 아니라, 미리 컨텍스트 윈도우에 끌어올 수 있게 해줌. 사용자 주도적 상호작용으로 사용자가 언제 가져올지 결정하고 모델을 기다릴 필요가 없어서 좋음.  
  
**Prompts의 활용 예시**  
  
> And I think similarly, you know, I wish, you know, more MCP servers today would bring prompts as examples of, like how to even use the tools. Yeah. at the same time.  
  
더 많은 MCP 서버들이 도구를 어떻게 사용하는지 예시로 프롬프트를 제공했으면 좋겠음.  
  
**Resources의 잠재력**  
  
> The resources bits are quite interesting as well. And I wish we would see more usage there because it's very easy to envision, but yet nobody has really implemented it. A system where like an MCP server exposes, you know, a set of documents that you have, your database, whatever you might want to as a set of resources. And then like a client application would build a full rack index around this, right?  
  
리소스 부분도 매우 흥미로움.  
  
구상하기는 쉽지만 실제로 구현한 사람은 없어서 더 많은 사용을 보고 싶음.  
  
MCP 서버가 문서나 데이터베이스 등을 리소스 세트로 노출하고, 클라이언트 애플리케이션이 이를 중심으로 전체 RAG 인덱스를 구축하는 시스템.  
  
> This is definitely an application use case we had in mind as to why these are exposed in such a way that they're not model driven, because you might want to have way more resource content than is, you know, realistically usable in a context window.  
  
컨텍스트 윈도우에서 현실적으로 사용할 수 있는 것보다 훨씬 많은 리소스 콘텐츠를 가질 수 있기 때문에 모델 주도가 아닌 방식으로 노출되도록 설계한 애플리케이션 사용 사례.  
  
---  
#### 그렇다면 tool 과 리소스를 어떻게 구별하는가?  
  
> The way we separate these is like tools are always meant to be initiated by the model.  
  
저희는 도구와 리소스를 이렇게 구분합니다. 도구는 항상 모델이 스스로 시작하도록 설계되어 있습니다.  
  
> It’s sort of like at the model’s discretion that it will find the right tool and apply it.  
  
모델의 재량에 따라 적절한 도구를 찾아 호출하는 흐름입니다.  
  
> So if that’s the interaction you want as a server developer, where it’s like, okay, this, you know, suddenly I’ve given the LLM the ability to run SQL queries, for example, that makes sense as a tool.  
  
따라서 서버 개발자로서 “이 LLM에게 SQL 쿼리를 실행할 권한을 주고, 스스로 판단해 쿼리하게 하고 싶다”면, 그건 분명히 도구(tool)로 구현하는 것이 맞습니다.  
  
> But resources are more flexible, basically.  
  
반면, 리소스(resource)는 훨씬 더 유연합니다.  
  
> And I think, to be completely honest, the story here is practically a bit complicated today. Because many clients don’t support resources yet.  
  
솔직히 오늘날 상황은 조금 복잡합니다. 아직 많은 클라이언트가 리소스를 지원하지 않기 때문이죠.  
  
> But like, I think in an ideal world where all these concepts are fully realized, and there’s like full ecosystem support, you would do resources for things like the schemas of your database tables and stuff like that, as a way to like either allow the user to say like, okay, now, you know, cloud, I want to talk to you about this database table. Here it is. Let’s have this conversation.  
  
하지만 이상적인 생태계를 상상해 보면, 데이터베이스 테이블의 스키마 같은 것들은 리소스로 제공해야 합니다. “자, 이 테이블에 대해 이야기해 보자”라며 URI 하나만 던지면, 모델이 알아서 그 리소스를 해석해 대화하도록 하는 식이죠.  
  
> Or maybe the particular AI application that you’re using, like, you know, could be something agentic, like cloud code, is able to just like agentically look up resources and find the right schema of the database table you’re talking about, like both those interactions are possible.  
  
혹은 여러분이 쓰는 AI 애플리케이션 자체가 에이전트 형태로, 백그라운드에서 리소스를 조회해 테이블 스키마를 찾아오는 것도 가능합니다.  
   
> But I think like, anytime you have this sort of like, you want to list a bunch of entities, and then read any of them, that makes sense to model as resources.  
   
일반적으로 여러 엔티티 목록을 다루고, 그중 하나를 읽어야 할 때는 리소스로 모델링하는 편이 자연스럽습니다.  
  
> Resources are also, they’re uniquely identified by a URI, always.  
  
리소스는 언제나 고유한 URI로 식별된다는 점도 특징입니다.  
  
> And so you can also think of them as like, you know, sort of general-purpose transformers, even like, if you want to support an interaction where a user just like drops a URI in, and then you like automatically figure out how to interpret that, you could use MCP servers to do that interpretation.  
  
따라서 일반적인 변환기(transformer)처럼도 사용할 수 있습니다. 사용자가 URI만 던져도 MCP 서버가 알아서 해석해 주는 시나리오를 구현할 수 있죠.  
  
> One of the interesting side notes here, back to the Z example of resources, is that has like a prompt library that you can do, that people can interact with. And we just exposed a set of default prompts that we want everyone to have as part of that prompt library.  
  
여기서 흥미로운 예시가 Zed 툴의 리소스 활용입니다. Zed는 MCP 서버를 통해 프롬프트 라이브러리를 자동으로 채우고, 기본 제공 프롬프트 세트를 불러오는 기능을 보여 주었죠.  
  
> Yeah, resources for a while so that like, you boot up Zed and Zed will just populate the prompt library from an MCP server, which was quite a cool interaction.  
  
즉 Zed를 실행하면 곧바로 MCP 서버에서 리소스를 가져와 프롬프트 라이브러리를 채우는 멋진 동작을 구현할 수 있었습니다.  
  
> And that was, again, a very specific, like, both sides needed to agree upon the URI format and the underlying data format.  
  
이 예시 역시 서버와 클라이언트 양쪽이 URI 포맷과 데이터 포맷에 합의했기에 가능했던 상호작용입니다.  
  
> And but that was a nice and kind of like neat little application of resources.  
  
정말 깔끔하고 흥미로운 리소스 활용 사례였습니다.  
  
> There’s also going back to that perspective of like, as an application developer, what are the things that I would want? Yeah. We also applied this thinking to like, you know, like, we can do this, we can do this, we can do this, we can do this. Like what existing features of applications could conceivably be kind of like factored out into MCP servers if you were to take that approach today.  
  
또 “애플리케이션 개발자로서 어떤 기능을 리소스로 떼어낼 수 있을까?”라는 관점으로도 확장할 수 있습니다. 예를 들어 IDE의 첨부 파일 메뉴 같은 기능은 자연스럽게 리소스로 모델링할 수 있죠.  
  
> And so like basically any IDE where you have like an attachment menu that I think naturally models as resources. It’s just, you know, those implementations already existed.  
  
결국 기존에 존재하던 여러 애플리케이션 기능을 MCP 서버의 리소스로 분리·운영할 수 있다는 점이 핵심입니다.  

>[!important]
>주요 내용
>**Tools vs Resources 구분 기준**
>- Tools: 모델이 자율적으로 판단하여 호출 (예: SQL 쿼리 실행)
>- Resources: 더 유연한 사용, URI로 식별, 사용자/애플리케이션이 제어 가능
>
>**실제 활용 예시**:
>- Sentry 백트레이스를 프롬프트로 가져오기
>- Zed 에디터의 프롬프트 라이브러리 자동 채우기
>- 데이터베이스 스키마를 리소스로 노출
>
>기존 애플리케이션 기능을 MCP 프리미티브로 재구성 가능

  
---  
#### 도구 호출 , 유연성 또는 제약  
  
> Thank you. Yeah. I mean, like, but, you know, I think for me as a developer relations person, I always insist on having a map for people. Here are all the main things you have to understand. We'll spend the next two hours going through this. So some one image that kind of covers all this, I think is pretty helpful. And I like your emphasis on prompts.  
> -swyx-  
  
감사합니다. 네. 제 입장에선 개발자 리레이션 담당자로서 사람들이 이해해야 할 핵심을 한눈에 보여주는 지도가 반드시 필요하다고 늘 강조해요. “다음 두 시간 동안 이것만 보면 됩니다”라는 식으로 말이죠. 그래서 이 모든 내용을 한 장에 담은 이미지 하나가 아주 유용하다고 생각합니다. 그리고 프롬프트 강조도 마음에 들어요.  
  
> And so I think prompts are not just single conversations. They're sometimes chains of conversations. Yeah.  
> -swyx-  
  
그래서 프롬프트는 단일 대화가 아니라 때로는 일련의 대화 시퀀스로 보는 것이 맞다고 생각합니다.  
  
>Another question that I had when I was looking at some server implementations, the server builders kind of decide what data gets eventually returned, especially for tool calls.  
>-Alessio-  
  
서버 구현체를 살펴볼 때 궁금했던 점은, 서버 빌더들이 도구 호출(tool call)에 대해 최종적으로 어떤 데이터를 반환할지 미리 결정해 둔다는 겁니다.  
  
>For example, the Google maps one, right? If you just look through it, they decide what, you know, attributes kind of get returned and the user can not override that if there's a missing one. That has always been my gripe with like SDKs in general, when people build like API wrapper SDKs. And then they miss one parameter that maybe it's new and then I can not use it.  
>-Alessio-  
  
예컨대 Google Maps 서버 래퍼를 보면, 어떤 속성(attributes)을 반환할지 고정해 둬서 사용자가 새로 추가된 필드를 사용할 수 없게 돼 있더군요. 저는 일반적으로 API 래퍼 SDK에서 이런 고정 반환 방식이 불만인데, 새 파라미터가 생겼는데 SDK가 이를 반영하지 않으면 그 기능을 전혀 못 쓰게 되니까요.  
  
>How do you guys think about that? And like, yeah, how much should the user be able to intervene in that versus just letting the server designer do all the work?  
  
여러분은 이 문제를 어떻게 보시나요? 서버 설계자가 모든 걸 결정하도록 맡길지, 사용자에게 얼마나 개입할 권한을 줘야 할지 궁금합니다.  
  
>I think we probably bear responsibility for the Google maps one, because I think that's one of the reference servers we've released.  
>- Justin/David -  
  
아마 Google Maps 예시는 저희가 공개한 참조 서버 중 하나라서 저희 책임이 크다고 봅니다.  
  
>I mean, in general, for things like for tool results in particular, we've actually made the deliberate decision, at least thus far, for tool results to be not like sort of structured JSON data, not matching a schema, really, but as like a text or images or basically like messages that you would pass into the LLM directly.  
  
일반적으로 도구 호출(tool result) 결과는 엄격한 JSON 스키마를 따르기보다, 모델에 직접 입력할 수 있는 메시지(텍스트나 이미지) 형태로 반환하도록 의도적으로 설계했습니다.  
  
>And so I guess the correlation that is, you really should just return a whole jumble of data and trust the LLM to like sort through it and see.  
  
그래서 “모든 데이터를 몽땅 반환하고, LLM이 알아서 필요한 정보를 골라 쓰게 하라”는 상관관계를 따르는 게 맞다고 생각합니다.  
  
>I mean, I think we've clearly done a lot of work. But I think we really need to be able to shift and like, you know, extract the information it cares about, because that's what that's exactly what they excel at.  
  
많은 노력을 기울였지만, LLM이 가장 잘하는 건 바로 중요한 정보를 추출하는 능력이기에, 그 강점을 믿어야 합니다.  
  
>And we really try to think about like, yeah, how to, you know, use LLMs to their full potential and not maybe over specify and then end up with something that doesn't scale as LLMs themselves get better and better.  
  
LLM이 점점 발전할수록 확장성 문제를 피하려면, 결과를 과도하게 규격화하지 않고 모델 잠재력을 최대한 활용하는 방향으로 가야 한다고 생각합니다.  
  
>So really, yeah, I suppose what should be happening in this example server, which again, will request welcome. It would be great. It's like if all these result types were literally just passed through from the API that it's calling, and then the API would be able to pass through automatically.  
  
결국 이 참조 서버 예시도, 호출하는 API의 원본 결과를 그대로 전달(passthrough)하도록 바뀌는 게 가장 좋을 것 같습니다.  
  
> And I think we're still need a little bit more relearning of how to build something for LLMs and trust them, particularly, you know, as they are getting significantly better year to year.  
  
LLM 성능이 해마다 크게 향상되는 만큼, LLM을 위한 설계 방식과 그 활용법을 다시 학습할 필요가 있다고 봐요.  
  
> Right. And I think, you know, two years ago, maybe that approach would have been very valid. But nowadays, just like just throw data at that thing that is really good at dealing with data is a good approach to this problem.  
  
맞아요. 2년 전 같았으면 이런 방식이 통하지 않았겠지만, 지금은 데이터 처리가 뛰어난 모델에 “모든 데이터를 던져라”가 좋은 해법이 될 수 있습니다.  
  
> And I think it's just like unlearning like 20, 30, 40 years of software engineering practices that go a little bit into this to some degree.  
  
결국 20~40년간 축적된 전통적 소프트웨어 엔지니어링 관행을 일부 잊어야 하는 셈이죠.  
  
> Like thinking, us thinking that like the biggest bottleneck to, you know, the next wave of capabilities for models might actually be their ability to like interact with the outside world to like, you know, read data from outside data sources or like take stateful actions.  
  
다음 세대 모델 역량의 최대 병목은 외부 데이터 소스를 읽거나 상태가 있는(stateful) 행동을 수행하는 능력이 될 수 있다는 점을 상기해야 합니다.  
  
> Working at Anthropic, we absolutely care about doing that. Safely and with the right control and alignment measures in place and everything.  
  
  
Anthropic에서도 바로 그 부분을 매우 중요하게 여기고 있습니다. 안전하고 관리·정렬(alignment) 장치를 갖춘 방식으로 말이죠.  
  
>[!Note]  
>alignment.
>
>AI alignment는 인공지능 시스템이 사람이나 집단의 의도된 목표, 선호도, 또는 윤리적 원칙을 향해 행동하도록 조종하는 것을 목표로 함 [AI alignment - Wikipedia](https://en.wikipedia.org/wiki/AI_alignment). 
>
>포괄적인 학술 조사논문에 따르면, AI alignment는 AI 시스템이 인간의 의도와 가치에 부합하게 행동하도록 만드는 것을 목표로 함 [[2310.19852] AI Alignment: A Comprehensive Survey](https://arxiv.org/abs/2310.19852).
>
>즉 Alignment 는 LLM의 출력이 인간의 의도와 가치를 반영하도록 조정하는 과정. 
>
>LLM은 본질적으로 확률을 기반으로 다음 단어를 예측하는 기계일 뿐이므로, 사전 학습된 LLM의 출력은 사용자의 의도나 인간의 가치와 잘 맞지 않을 수 있음.

>[!Note]
>**RLHF(Reinforcement Learning from Human Feedback)**
>
>alignment를 그래서 어떻게 하는데? 의 대답.
>OpenAI가 ChatGPT를 만들 때 사용한 방법으로, 인간의 피드백을 바탕으로 강화 학습을 수행하여 인간의 가치를 LLM에 주입하는 방법.

>[!Note]  
>alignment tax
>
>LLM의 신뢰성과 안전성을 확보하기 위해 정렬은 필수적이지만, 한 가지 문제가 존재함. 
>
>일반적으로 정렬 과정에서 LLM의 성능이 떨어지게됨. 
>
>사실 그럴 수밖에 없는 게, 잠재적인 문제를 야기할 수 있는 질문에 대한 응답을 거부하다 보면, LLM이 제공하는 정보가 제한될 수밖에 없음. 
>
>이처럼 정렬 과정에서 LLM의 성능이 저하되는 현상을 정렬 비용(Alignment Tax)이라고 함

  
> But also as AI gets better, people will want that. That'll be key to like becoming productive with AI is like being able to connect them up to all those things.  
  
  
AI가 발전할수록 사용자들은 외부 시스템 연동을 원할 것이고, 생산성을 위해 필수 기능이 될 것입니다.  
  
> So MCP is also sort of like a bet on the future and where this is all going and how important that will be.  
  
  
따라서 MCP는 바로 그 미래—AI와 외부 세계 간의 연결이 얼마나 중요한지를 베팅한 설계라고 볼 수 있습니다.  
  
---  
  
### MCP vs OpenAPI  
  
>MCP vs OpenAPI: 이게 결론입니다”라고 말할 수 있는 명쾌한 설명을 부탁드리고 싶었어요.  
>-swyx-  
  
>Yeah, I think fundamentally, I mean, open API specifications are a very great tool. And like I've used them a lot in developing APIs and consumers of APIs. I think fundamentally, or we think that they're just like too granular for what you want to do with LLMs. Like they don't express higher level AI specific concepts like this whole mental model. Yeah.  
>- Justin/David -  
  
네, 기본적으로 OpenAPI 사양은 훌륭한 도구입니다. 저도 API를 설계하고 소비하는 데 많이 사용해 왔죠. 다만 LLM 기반 애플리케이션에서는 너무 세부적(granular)이라, AI 고유의 상위 개념(mental model)을 담아내기 어렵다고 봅니다.  
  
>But we've talked about with the primitives of MCP and thinking from the perspective of the application developer, like you don't get any of that when you encode this information into an open API specification. So we believe that models will benefit more from like the purpose built or purpose design tools, resources, prompts, and the other primitives than just kind of like, here's our REST API, go wild.  
  
하지만 MCP의 primitives —도구, 리소스, 프롬프트 등을 애플리케이션 개발자 관점에서 설계하면, OpenAPI에 이 정보를 담아내는 것만으로는 부족합니다. 따라서 REST API “그대로 던져 주세요” 접근법보다는, AI 목적에 특화된 원시요소를 사용하는 편이 모델 성능에 더 이롭다고 믿습니다.  
  
>I do think there, there's another aspect. I think that I'm not an open API expert, so I might, everything might not be perfectly accurate. But I do think that we're... Like there's been, and we can talk about this a bit more later. There's a deliberate design decision to make the protocol somewhat stateful because we do really believe that AI applications and AI like interactions will become inherently more stateful and that we're the current state of like, like need for statelessness is more a temporary point in time that will, you know, to some degree that will always exist.  
  
또 한 가지 중요한 점은, MCP 프로토콜을 일부 상태(stateful) 기반으로 설계했다는 겁니다. 제가 OpenAPI 전문가가 아니라 완벽치 않을 수 있지만, AI 애플리케이션 상호작용이 점점 상태를 유지하는 방향으로 진화할 것이라고 보기 때문입니다. 현재의 무상태(stateless) 모델은 일시적이며, 결국 상태 관리가 필수가 될 것입니다.  
  
>But I think like more statefulness will become increasingly more popular, particularly when you think about additional modalities that go beyond just pure text-based, you know, interactions with models, like it might be like video, audio, whatever other modalities exist and out there already.  
  
특히 텍스트를 넘어 비디오·오디오 등 다양한 입력·출력 모달리티가 결합될수록, 상태 관리가 더욱 중요해질 것입니다.  
  
>And so I do think that like having something a bit more stateful is just inherently useful in this interaction pattern.  
  
따라서 상태(stateful)를 유지하는 설계가 이런 상호작용 패턴에서 본질적으로 유용하다고 봅니다.  
  
>One more thing to add here is that we've already seen people, I mean, this happened very early. People in the community built like bridges between the two as well. So like, if what you have is an open API specification and no one's, you know, building a custom MCP server for it, there are already like translators that will take that and re-expose it as MCP. And you could do the other direction too. Awesome.  
  
추가로, 커뮤니티에서는 이미 양방향 변환 브리지를 만든 사례가 있습니다. OpenAPI 사양을 MCP 서버로 중계(translator)하거나, 반대로 MCP 사양을 OpenAPI로 내보내는 도구가 개발되어 있죠.  

>[!important]
>주요 내용
>**도구 결과 반환 철학**
>- 엄격한 스키마보다 유연한 메시지 형태 선호
>- LLM의 정보 추출 능력을 신뢰
>- API 원본 데이터를 그대로 전달(passthrough)
>
**MCP vs OpenAPI 차이점**:
>- OpenAPI: 너무 세부적(granular), REST API 중심
>- MCP: AI 특화 상위 개념, 상태유지(stateful) 설계
>- 미래 멀티모달 상호작용 대비
>
> 커뮤니티에서 양방향 브리지 개발 중
  
---  
### MCP 서버를 구축하는 것에 대해  
  
>So I think everybody does the tweets about like connect the cloud desktop to XMCP. It’s amazing. How would you guys suggest people start with building servers? I think the spec is like, so there’s so many things you can do that. It’s almost like, how do you draw the line between being very descriptive as a server developer versus like going back to our discussion before, like just take the data and then let them auto manipulate it later. Do you have any suggestions for people?  
  
모두 클라우드 데스크톱을 MCP에 연결하는 트윗만 하지만, 실제로는 서버를 어떻게 시작해야 할지 모르잖아요. 스펙만 봐도 할 수 있는 게 너무 많아서, 서버 개발자로서 얼마나 상세하게 작성할지, 아니면 “데이터만 던지고 모델에게 맡길지” 사이의 선을 어떻게 정해야 할지 고민이 큽니다. 개발자들에게 어떤 조언을 해 주시겠어요?  
  
>And so I think that the best part is just like pick the language of, you know, of your choice that you love the most, pick the SDK for it, if there’s an SDK for it, and then just go build a tool of the thing that matters to you personally. And that you want to use. You want to see the model like interact with, build the server, throw the tool in, don’t even worry too much about the description just yet, like do a bit of like, write your little description as you think about it and just give it to the model and just throw it to standard IO protocol transport wise into like an application that you like and see it do things.  
>- Justin/David -  
  
가장 좋은 방법은, 여러분이 가장 편한 언어와 SDK를 선택한 뒤, 개인적으로 필요한 기능을 도구(tool)로 구현해 보는 겁니다. 상세한 설명문(descriptions)은 나중에 다듬어도 돼요. 일단 간단히 설명만 작성하고 모델에게 전달해 보세요. 표준 입출력(Standard IO) 방식으로 애플리케이션에 연결한 뒤, 모델이 어떻게 반응하는지 체험해 보는 게 중요합니다.  
  
>And I think that’s part of the magic that, or like, you know, empowerment and magic for developers to get so quickly to something that the model does. Or something that you care about. That I think really gets you going and gets you into this flow of like, okay, I see this thing can do cool things. Now I go and, and can expand on this and now I can go and like really think about like, which are the different tools I want, which are the different raw resources and prompts I want. Okay. Now that I have that. Okay. Now do I, what do my evals look like for how I want this to go? How do I optimize my prompts for the evals using like tools like that? This is infinite depth so that you can do. But. Okay. Just start. As simple as possible and just go build a server in like half an hour in the language of your choice and how the model interacts with the things that matter to you. And I think that’s where the fun is at.  
  
이런 방식이야말로 개발자를 빠르게 “모델이 이런 것도 할 수 있구나” 하는 경험으로 이끌어 줍니다. 그러면 자연스럽게 “다음엔 어떤 도구와 리소스, 프롬프트를 추가해야 할까?”, “평가(evals)는 어떻게 설계하지?”, “프롬프트를 어떻게 최적화하지?” 같은 고민으로 흘러가게 되고요. 할 수 있는 일은 무한합니다. 일단 최대한 간단하게 시작해 보세요. 반시간 안에 자신이 원하는 언어로 서버를 만들어 보고, 모델과 상호작용해 보면 재미가 느껴질 겁니다.  
  
>I also, I’m quite partial, again, to using AI to help me do the coding. Like, I think even during the initial development process, we realized it was quite easy to basically just take all the SDK code. Again, you know, what David suggested, like, you know, pick the language you care about, and then pick the SDK. And once you have that, you can literally just drop the whole SDK code into an LLM’s context window and say, okay, now that you know MCP, build me a server that does that. This, this, this.  
  
저는 코딩 자체를 AI에 맡기는 방법도 적극 추천합니다. 초기 개발 단계에서, 선택한 언어와 SDK 코드를 LLM 컨텍스트에 그대로 넣고 “이제 MCP 기반으로 이 기능을 구현해 줘”라고 요청해 보세요. 놀라울 만큼 그럴듯한 코드가 나옵니다.  
  
> Yeah. And if you don’t have an SDK, again, give the subset of the spec that you care about to the model, and like another SDK and just have it build you an SDK. And it usually works for like, that subset. Building a full SDK is a different story. But like, to get a model to tool call in Haskell or whatever, like language you like, it’s probably pretty straightforward.  
  
만약 SDK가 없다면, 관심 있는 스펙 일부를 모델에 알려주고 “이 부분만 처리하는 SDK를 만들어 줘”라고 요청해 보세요. 그 정도면 충분히 동작합니다. 전체 SDK를 완성하는 건 별도 작업이지만, 특정 언어로 도구 호출 코드(tool call)를 받는 건 상당히 쉬워요.  
  
....  
  
> And I think you see some of that in the community already, but there's just, you know, things like, Hey, summarize my, you know, my, my, my, my favorite subreddits for the morning MCP server that nobody has built yet, but it's very easy to envision. And the protocol can totally do this. And these are like slightly richer experiences.  
  
커뮤니티에서도 이미 일부 사례를 볼 수 있지만, 예를 들어 “내가 구독한 좋아하는 서브레딧 내용을 매일 아침 요약해 줘” 같은 MCP 서버는 아직 없지만, 충분히 상상 가능하고 프로토콜이 완벽히 지원할 수 있습니다.  
  
>And I think as people like go away from like the, oh, I just want to like, I'm just in this new world where I can hook up the things that matter to me, to the LLM, to like actually want a real workflow, a real, like, like more richer experience that I, I really want exposed to the model. I think then you will see these things pop up, but again, that's a, there's a little bit of a chicken and egg problem at the moment with like what a client supported versus, you know, what servers like authors want to do. Yeah.  
  
사람들이 “관심 있는 시스템을 LLM에 연결만 하면 된다” 수준을 넘어, 실제로 의미 있는 워크플로우와 더 풍부한 사용자 경험을 구현하고 싶어질 때, 이런 새로운 MCP 서버들이 속속 등장할 겁니다. 다만 지금은 클라이언트 지원과 서버 개발자 의도가 맞물려야 하는 약간의 닭이 먼저냐 달걀이 먼저냐 문제가 있죠.  

---
### Composability  
  
>That, that, that was. That's kind of my next question on composability. Like how, how do you guys see that? Do you have plans for that? What's kind of like the import of MCPs, so to speak, into another MCP? Like if I want to build like the subreddit one, there's probably going to be like the Reddit API, uh, MCP, and then the summarization MCP. And then how do I, how do I do a super MCP?  
>- Alessio -  
  
바로 그 점이 제 다음 질문, ‘컴포저빌리티(composability)’인데요. MCP 서버끼리 서로 연동하는 건 어떻게 보시나요? 예컨대 “서브레딧 요약” 서버를 만들려면, 레딧 API MCP와 요약 MCP를 각각 구축해야 할 텐데, 이 둘을 합쳐 ‘슈퍼 MCP’는 어떻게 구성하나요?  
  
>Yeah. So, so this is an interesting topic and I think there, um, so there, there are two aspects to it.  
>- Justin/David -  
  
네, 흥미로운 주제입니다. 이에는 두 가지 측면이 있다고 생각해요.  
  
> I think that the one aspect is like, how can I build something? I think agentically that you requires an LLM call and like a one form of fashion, like for summarization or so, but I’m staying model independent and for that, that’s where like part of this by directionality comes in, in this more rich experience where we do have this facility for servers to ask the client again, who owns the LLM interaction, right?  
  
한 측면은 “어떻게 구축할 것인가?”입니다. 요약 같은 작업은 LLM 호출이 필요하겠지만, 저는 모델 독립성을 유지하고 싶어요. 이때 ‘양방향성(directionality)’이 중요한데, 서버가 “LLM과의 상호작용을 담당하는 클라이언트에게 다시 요청할 수 있는” 기능이죠.  
  
> Like we talk about cursor, who like runs the, the, the loop with the LLM for you there that for the server author to ask the client for a completion. Um, and basically have it like summarize something for the server and return it back.  
  
예컨대 Cursor는 LLM과의 대화 루프를 클라이언트를 통해 실행합니다. 서버 작성자가 클라이언트에게 완성(completion)을 요청하면, 클라이언트가 응답을 요약해 서버로 반환하죠.  
  
> And so now what model summarizes this depends on which one you have selected in cursor and not depends on what the author brings. The author doesn’t bring an SDK. It doesn’t have, you had an API key. It’s completely model independent, how you can build this.  
  

따라서 어떤 모델이 요약할지는 Cursor에서 선택한 모델에 달려 있고, 서버 작성자가 직접 SDK나 API 키를 제공할 필요가 없습니다. 완전히 모델 독립적으로 구성할 수 있죠.  
  
> There’s just one aspect to that. The second aspect to building richer, richer systems with MCP is that you can easily envision an MCP server that serves something to like something like cursor or win server for a cloud desktop, but at the same time, also is an MCP client at the same time and itself can use MCP servers to create a rich experience.  
  
그리고 두 번째 측면은, 단일 MCP 서버가 클라우드 데스크톱용 Cursor나 Win Server에 서비스를 제공하는 동시에, 스스로도 MCP 클라이언트로 동작하며 또 다른 MCP 서버를 호출해 풍부한 경험을 만들어 낼 수 있다는 겁니다.  
  
> And now you have a recursive property, which we actually quite carefully in the design principles, try to retain. You know, you see it all over the place and authorization and other aspects, to the spec that we retain this like recursive pattern.  
  
이로써 재귀적(recursive) 특성이 생깁니ㅌ다. 설계 원칙에도 재귀 패턴을 유지하려고 신경 썼고, 권한 관리 등 스펙 전반에서 이 패턴을 확인할 수 있죠.  
  
> And now you can think about like, okay, I have this little bundle of applications, both a server and a client. And I can add. Add these in chains and build basically graphs like, uh, DAGs out of MCP servers that can just richly interact with each other.  
  
이제 “서버이자 클라이언트인 작은 애플리케이션 묶음”을 생각해 보세요. 이들을 체인으로 연결해 DAG(Directed Acyclic Graph) 형태의 네트워크를 구성하면 MCP 서버들이 서로 풍부하게 상호작용할 수 있습니다.  
  
> A agentic MCP server can also use the whole ecosystem of MCP servers available to themselves. And I think that’s a really cool environment, cool thing you can do. And people have experimented with this.  
  
에이전트형 MCP 서버는 자신이 접근 가능한 모든 MCP 서버 생태계를 활용할 수 있습니다. 정말 멋진 환경이고, 이미 여러 실험이 진행되었죠.  
  
> And I think you see hopefully more of this, particularly when you think about like auto-selecting, auto-installing, there’s a bunch of these things you can do that make, make a really fun experience.  
  
자동 선택(auto-selecting)·자동 설치(auto-installing) 같은 기능까지 더해지면, 더욱 흥미로운 경험이 가능할 겁니다.  
  
>[!note]  
>auto-selecting, auto-installing 
>특정 기능이 필요한 시점에 자동으로 mcp 서버를 선택하거나,  mcp 서버를 설치하는것을 의미하는것 같음.
  
> I, I think practically there are some niceties we still need to add to the SDKs to make this really simple and like easy to execute on like this kind of recursive MCP server that is also a client or like kind of multiplexing together the behaviors of multiple MCP servers into one host, as we call it.  
  
실제로 이런 재귀형 MCP 서버(클라이언트 겸용)나 여러 서버 동작을 하나의 호스트에서 복합 실행(multiplexing)하려면, SDK에 몇 가지 편의 기능을 더해야 합니다.  
  
> These are things we definitely want to add. We haven’t been able to yet, but like, I think that would go some way to showcasing these things that we know are already possible, but not necessarily taken up that much yet.  
  
이 기능들은 꼭 추가하고 싶지만 아직 미완성입니다. 하지만 구현되면 가능성을 보여 주는 좋은 사례가 될 것입니다.  
  
> This is, uh, very exciting. And very, I'm sure, I'm sure a lot of people get very, very, uh, a lot of ideas and inspiration from this. Is an MCP server that is also a client, is that an agent?  
> -swyx-  
  
정말 흥미롭네요. 많은 분이 영감과 아이디어를 얻을 것 같은데, “서버이자 클라이언트인” MCP 서버를 에이전트라고 부를 수 있나요?  
  
>What's an agent? There's a lot of definitions of agents.  
>- Justin/David-  
  
에이전트가 정확히 뭔가요? 정의가 다양하죠.  
  
  
>Because like you're, in some ways you're, you're requesting something and it's going off and doing stuff that you don't necessarily know. There's like a layer of abstraction between you and the ultimate raw source of the data. You could dispute that. Yeah. I just, I don't know if you have a hot take on agents.  
>-swyx-  
  
사용자가 요청하면, 그 뒤에서 모르는 작업을 수행하잖아요. 데이터의 최종 원천과 사용자 사이에 추상화 계층이 생기고요. 이걸 에이전트라고 볼 수도 있겠죠. 에이전트에 대한 의견이 있나요?  
  
> I do think, I do think that you can build an agent that way. For me, I think you need to define the difference between an MCP server plus client that is just a proxy versus an agent. I think there's a difference.  
> -Justin/David-  
  
네, 그렇게 에이전트를 구성할 수 있다고 봅니다. 다만 “단순 프록시(proxy)로서 서버+클라이언트”와 “에이전트”를 구분해야 합니다. 분명 차이가 있어요.  
  
> And I think that difference might be in, um, you know, for example, using a sample loop to create a more richer experience to, uh, to, to have a model call tools while like inside that MCP server through these clients. I think then you have a, an actual like agent.  
  
예를 들어 샘플 루프(sample loop)를 사용해 모델이 MCP 서버 내부에서 도구 호출을 반복하며 더욱 풍부한 경험을 제공한다면, 그때가 진정한 에이전트가 아닐까 싶습니다.  
  
> Yeah. I do think it's very simple to build agents that way. Yeah. I think there are maybe a few paths here. Like it definitely feels like there's some relationship. Between MCP and agents.  
  
네, 그렇게 에이전트를 구축하는 건 꽤 간단합니다. MCP와 에이전트 간에는 분명한 연관성이 있다고 봐요.  
  
> One possible version is like, maybe MCP is a great way to represent agents. Maybe there are some like, you know, features or specific things that are missing that would make the ergonomics of it better. And we should make that part of MCP. That's one possibility.  
  
한 가지 방안은 MCP 자체를 에이전트 표현 방식으로 활용하는 것입니다. 부족한 기능이 있다면 MCP에 추가해도 되고요.  
  
> Another is like, maybe MCP makes sense as kind of like a foundational communication layer for agents to like compose with other agents or something like that.  
  
또 다른 방안은, MCP를 에이전트들이 서로 조합(compose)할 수 있는 기본 통신 계층으로 쓰는 것입니다.  
   
> Or there could be other possibilities entirely. Maybe MCP should specialize and narrowly focus on kind of the AI application side. And not as much on the agent side. I think it's a very live question and I think there are sort of trade-offs in every direction going back to the analogy of the God box.  
  
혹은 MCP가 AI 애플리케이션 쪽에만 집중하고, 에이전트 기능은 다른 프로토콜에 맡길 수도 있겠죠. 이는 아직 열린 질문이며, 만능(God box) 접근의 장단점을 저울질해야 합니다.  
  
> I think one thing that we have to be very careful about in designing a protocol and kind of curating or shepherding an ecosystem is like trying to do too much. I think it's, it's a very big, yeah, you know, you don't want a protocol that tries to do absolutely everything under the sun because then it'll be bad at everything too.  
  
프로토콜을 설계하고 생태계를 조율할 때 가장 조심해야 할 건, 지나치게 많은 기능을 한 번에 넣으려는 것입니다. 모든 걸 다 하려다 보면, 결국 어느 것에도 능숙하지 못해집니다.  
  
> And so I think the key question, which is still unresolved is like, to what degree are agents really naturally fitting in to this existing model and paradigm or to what degree is it basically just like orthogonal? It should be something.  
  
따라서 핵심 질문은 “에이전트가 MCP 모델·패러다임에 자연스럽게 녹아드는가?” 아니면 “전혀 별개(orthogonal)로 다뤄야 하는가?” 입니다. 이 부분은 아직 결론이 나지 않았습니다.  

**핵심 요약:**

>[!important]
>주요 내용
>**MCP 서버 구축 시작 방법**:
>- 좋아하는 언어와 SDK 선택
>- 개인적으로 필요한 기능부터 구현
>- 30분 안에 간단한 서버 만들어보기
>- AI를 활용한 코딩 추천
>
>**컴포저빌리티(Composability)**:
>- 서버가 동시에 클라이언트 역할 가능
>- 재귀적 패턴으로 MCP 서버 체인 구성
>- DAG 형태의 네트워크 구축 가능
>
>**MCP와 에이전트의 관계**:
>- 아직 명확히 정의되지 않은 영역
>- MCP가 에이전트 표현 방식이 될 수도 있고
>- 에이전트 간 통신 계층이 될 수도 있음

---  
### 도구 사용 개수와 제어 권한 관리  
  
>Anyway, so the first one is, it's just a simple, how many MCP things can one implementation support, you know, so this is the, the, the sort of wide versus deep question.  
>-swyx-  
  
좋습니다. 좀 더 구체적인 질문으로 넘어가 보죠. 첫 번째는 단순합니다. “하나의 구현에서 얼마나 많은 MCP 서버(또는 도구·리소스·프롬프트 등)를 지원할 수 있느냐?”라는 폭(많이) 대 깊이(적게) 문제입니다.  

>And, and this, this is direct relevance to the nesting of MCPs that we just talked about in April, 2024, when, when Claude was launching one of its first contexts, the first million token context example, they said you can support 250 tools.

그리고 이 질문은 2024년 4월, Claude가 첫 번째 컨텍스트(백만 토큰 맥락) 예시를 발표하며 “250개의 도구를 지원할 수 있다”고 말했던 MCP 중첩(nesting) 문제와 직접적으로 연관됩니다.

>And in a lot of cases, you can't do that.  

하지만 실제로는 그 정도를 지원하지 못하는 경우가 많습니다.

> You know, so to me, that's wide in, in the sense that you, you don't have tools that call tools.  

제게 있어 ‘넓다(wide)’는 의미는, 도구(tool)가 다른 도구를 호출하지 않는다는 뜻입니다.

> You just have the model and a flat hierarchy of tools, but then obviously you have tool confusion.  

 단순히 모델과 평면적(flat)인 도구 계층(hierarchy)이 있을 뿐인데, 이 경우 도구 간 혼선(confusion)이 발생하기 마련입니다.

> It's going to happen when the tools are adjacent, you call the wrong tool.  

 도구들이 서로 가까이(adjacent) 있을 때, 잘못된 도구를 호출하는 상황이 발생할 수밖에 없습니다.

> You're going to get the bad results, right?  

그럼 당연히 원하는 결과가 나오지 않겠죠?

> Do you have a recommendation of like a maximum number of MCP servers that are enabled at any given time?  

어떤 시점에 활성화할 MCP 서버의 **최대 개수**에 대해 권장하는 바가 있나요?

 >I think be honest, like, I think there's not one answer to this because to some extent, it depends on the model that you're using.  
 -Justin/David-

솔직히 말씀드리자면, 이 질문에는 정답이 하나만 있는 게 아닙니다. 어느 정도는 사용 중인 **모델(model)**에 달려 있기 때문이죠.

>[!Note]
>결국 모델 성능, 모델에서 제공하는 정보, 피쳐에 달려있음.

> I mean, I think that the dream is certainly like you just furnish all this information to the LLM and it can make sense of everything.  

꿈꾸는 이상(ideal)은 이런 모든 정보를 LLM에 제공하면, LLM이 알아서 모든 것을 해석해 주는 것입니다.

> This, this kind of goes back to like the, the future we envision with MCP is like all this information is just brought to the model and it decides what to do with it.  

이것은 MCP의 미래 청사진과도 연결되는데, 모든 정보가 모델에게 제공되고 **모델이 스스로** 처리 방안을 결정하는 구조를 지향한다는 뜻이죠.

> But today the reality or the practicalities might mean that like, yeah, maybe you, maybe in your client application, like the AI application, you do some fill in the blanks.  

그러나 현재 현실적으로는, 클라이언트 애플리케이션(또는 AI 애플리케이션) 쪽에서 **빈칸 보완(fill in the blanks)** 작업을 해야 할 수도 있습니다.

> Maybe you do some filtering over the tool set or like maybe you, you run like a faster, smaller LLM to like filter to what's most relevant and then only pass those tools to the bigger model.  

도구 세트를 **필터링(filtering)** 하거나, 더 빠르고 작은 LLM을 사용해 가장 관련성 높은 도구만 선별한 뒤, 그 도구들만 더 큰 모델에 전달하는 식으로 말이죠.

> Or you could use an MCP server, which is a proxy to other MCP servers and does some filtering at that level or something like that.  

또는 MCP 서버를 **프록시(proxy)** 로 두어, 그 서버에서 다른 MCP 서버들을 필터링하는 단계(routing/filtering)를 구현할 수도 있습니다.

>I think hundreds, as you referenced, is still a fairly safe bet, at least for Claude.  

제가 보기에는, 적어도 Claude의 경우 **수백 단위(hundreds)** 는 여전히 안전한 선택입니다.

#### description and overlap

> Yeah, and obviously it highly, it highly depends on the overlap of the description, right?  

네, 그리고 분명히 **도구 설명(description)의 중복(overlap)** 여부에 크게 달려 있습니다.

> Like if you, if you have like very separate servers that do very separate things and the tools have very clear unique names, very clear, well-written descriptions, you know, your mileage might be more higher than if you have a GitLab and a GitHub server at the same time in your context.  

예컨대, 완전히 분리된 기능을 수행하는 서버들이 있고, 그 도구들의 이름이 명확하게 고유(unique)하며, 설명이 잘 작성되어 있다면, GitLab과 GitHub 같은 유사한 도구를 동시에 둔 경우보다 훨씬 더 성능이 좋을 수 있습니다.

> And, and then the overlap is quite significant because they look very similar to the model and confusion becomes easier.  

반면, 둘의 설명이 많이 겹치면 모델 입장에서 구분이 어려워져 혼선이 더 쉽게 발생하겠죠.

> Depending on the AI application, if you're, if you're trying to build something very agentic, maybe you are trying to minimize the amount of times you need to go back to the user with a question or, you know, minimize the amount of like configurability in your interface or something.  

에이전트형(agentic) 애플리케이션을 만들려면, 사용자에게 되묻는 횟수를 최소화하거나, 인터페이스의 설정(configurability)을 최대한 줄이는 방향으로 설계할 수도 있겠죠.

> But if you're building other applications, you're building an IDE or you're building a chat application or whatever, like, I think it's totally reasonable to have affordances that allow the user to say like, at this moment, I want this feature set or at this different moment, I want this different feature set or something like that.  

반면 IDE나 채팅 애플리케이션을 만든다면, “지금은 이 기능 세트를 쓰고 싶다”, “다른 순간에는 저 기능 세트를 쓰고 싶다”라고 사용자가 직접 선택할 수 있는 **UI 어포던스(affordances)** 를 제공하는 것도 충분히 합리적입니다.

> And maybe not treat it as like always on.  

그리고 항상 모든 기능이 켜진 상태(always-on)로 두지 않아도 될 겁니다.

> The full list always on all the time.  

전체 도구 목록이 항상 활성화된 상태일 필요는 없으니까요.

> I guess the way I think about this is still like at the end of the day, and this is a core MCP design principle is like, ultimately, the concept of a tool is not a tool.  

제 관점은 결국, 그리고 이것이 핵심 MCP 설계 원칙 중 하나인데, “도구(tool)라는 개념 자체가 도구가 아니다”라는 것입니다.

> It's a client application, and by extension, the user.  

즉 도구는 **클라이언트 애플리케이션**이고 확장해서 말하면 결국 **사용자(user)** 인 셈이죠.

> Ultimately, they should be in full control of absolutely everything that's happening via MCP.  

궁극적으로 MCP를 통해 일어나는 모든 일에 대해 사용자가 **완전한 제어(full control)**권을 가져야 합니다.

> When we say that tools are model controlled, what we really mean is like, tools should only be invoked by the model.  

 “도구가 모델에 의해 제어된다(model-controlled)”고 할 때 진짜 의미는 “도구는 오직 모델에 의해서만 호출(invoked)되어야 한다”는 뜻입니다.

> Like there really shouldn't be an application interaction or a user interaction where it's like, okay, as a user, I now want you to use this tool.  

예를 들어 사용자가 “자, 이 도구를 사용해줘”라고 명령하는 **UI 상의 인터랙션**은 없어야 한다는 거죠.

> I mean, occasionally you might do that for prompting reasons, but like, I think that shouldn't be like a UI affordance.  

프롬프트(prompting)를 위해 가끔 그렇게 할 순 있겠지만, 그것이 **일상적인 UI 어포던스**가 되어서는 안 됩니다.

> But I think the client application or the user deciding to like filter out the user, it's not a tool.  

그러나 클라이언트 애플리케이션이나 사용자가 도구 목록에서 특정 항목을 필터링(filter out)하는 것은 도구가 아닙니다.

> I think the client application or the user deciding to like filter out things that MCP servers are offering, totally reasonable, or even like transform them.  

MCP 서버가 제공하는 도구를 **필터링**하거나 **변환(transform)** 하는 것은 전적으로 합리적이라고 봅니다.

> Like you could imagine a client application that takes tool descriptions from an MCP server and like enriches them, makes them better.  

예를 들어 MCP 서버로부터 받아온 도구 설명을 **강화(enrich)** 하여 더 나은 설명으로 바꾸는 애플리케이션을 상상해 볼 수 있겠죠.

> We really want the client applications to have full control in the MCP paradigm.  

우리는 MCP 패러다임 내에서 클라이언트 애플리케이션이 **전권(full control)** 을 갖기를 원합니다.

> That in addition, though, like I think there, one thing that's very, very early in my thinking is there might be a addition to the protocol where you want to give the server author the ability to like logically group certain primitives together, potentially.  

다만 제 초기 구상 중 하나는, 프로토콜에 **서버 저자(author)**가 특정 프리미티브(primitives)를 논리적으로 묶을 수 있는 기능을 추가하는 것입니다.

> To inform that, because they might know some of these logical groupings better, and that could like encompasses prompts, resources, and tools at the same time.  

왜냐하면 서버 저자가 특정 도구 묶음이나 프롬프트, 리소스를 더 잘 이해하고 있을 수 있기 때문입니다.

> I mean, personally, we can have a design discussion on there.  

 개인적으로는 그 부분에 대해 디자인 논의를 해볼 수 있다고 생각합니다.

> I mean, personally, my take would be that those should be separate MCP servers, and then the user should be able to compose them together.  

제 개인적 의견은, 그런 묶음은 **별도의 MCP 서버**로 두고, 사용자가 그들 간을 **조합(compose)**할 수 있어야 한다는 것입니다.

> Is there going to be like a MCP standard library, so to speak, of like, hey, these are like the canonical servers, do not build this.  
> -Alessio-

 일종의 **MCP 표준 라이브러리(standard library)**가 제공되어, “여기 있는 게 공인된(canonical) 서버들이다. 직접 만들 필요 없다”라는 방식이 될까요?

> We're just going to take care of those.  

 우리가 그 목록을 관리해 주고요.

> And those can be maybe the building blocks that people can compose.  

 그것들이 사람들이 조합해 쓸 수 있는 **빌딩 블록(building blocks)** 이 될 수 있겠지요.

 >I think we will not be prescriptive in that sense.  
 >-Justin/David-
 
그 점에 대해서는 우리가 **규격화(prescriptive)** 하지 않을 것입니다.

---
### 안전하고 믿을만한 구현체를 어떻게 판단 하는가

open source의 특성상 여러개의 구현체가 만들어지고 몇몇 구현체로 수렴하는 구조로 간다면.
안전하고 믿을만한 구현체를 어떻게 판단할수 있는가?

>Like, how do you determine which MCP servers are like the kind of good and safe ones to use?

어떤 MCP 서버가 “안전하고 믿을 만한 구현체”인지 어떻게 판단할 것인가가 관건입니다.

> But you want to make sure that you're not using ones that are really like sus, right?  

정말 **수상(sus)** 쩍은 구현체를 쓰지 않도록 주의해야 합니다.

> And so trying to think about like how to kind of endow reputation or like, you know, if hypothetically.  

그래서 어떻게 하면 구현체에 **평판(reputation)** 을 부여하거나, 예컨대,

> Anthropic is like, we've vetted this.  

“Anthropic에서 검증(vetted)했다”라는 식의

> It meets our criteria for secure coding or something.  

 “우리의 보안 코딩 기준을 충족한다”라는

> How can that be reflected in kind of this open model where everyone in the ecosystem can benefit?  

 표준화된 방식으로 반영해서 에코시스템 전체가 혜택을 볼 수 있도록 할 수 있을까요?

> Don't really know the answer yet, but that's very much top of mind.  

아직 정답은 모르겠지만, 현재 이 부분을 가장 중점적으로 고민 중입니다.

> And a registry is very tempting to offer download counts, likes, reviews, and some kind of trust thing.  
> -swyx-
 
레지스트리는 다운로드 수, 좋아요, 리뷰 같은 **소셜 증명(social proof)**을 제공하기에 매력적인 플랫폼이죠.

> I think it's kind of brittle.  

 하지만 저는 그 시스템이 **취약(brittle)** 하다고 봅니다.

> Like no matter what kind of social proof or other thing you can, you can offer, the next update can compromise a trusted package.  

어떤 형태의 소셜 증명이라도, 다음 업데이트 때 **신뢰받는 패키지(trusted package)** 가 훼손될 위험이 있기 때문입니다.

>So abusing the trust system is like setting up a trust system creates the damage from the trust system.  

즉, **신뢰 시스템**을 악용(abuse)하면, 신뢰 시스템 자체가 오히려 피해를 낳습니다.

> Yeah, absolutely. Cool.  
> -Justin/David-

네, 전적으로 동의합니다. 멋지네요.

> And then I think like that's very classic, just supply chain problem that like all registries effectively have.  

 이것은 사실 모든 레지스트리가 갖는 전형적인 **공급망 문제(supply chain problem)** 이기도 합니다.

---
### 메모리기능 의 simple 서버 사용 권장

> And I think I really, really encourage people should look at these, what I call special servers.  

제가 **스페셜 서버(special servers)**라고 부르는 것들을 꼭 보시길 적극 권장합니다.

> Like they're, they're not normal servers in the, in the sense that they, they wrap some API and it's just easier to interact with those than to work at the APIs.  

일반 서버가 아니라, **API 래핑(wrap)** 을 통해 API를 직접 다루는 것보다 훨씬 **간편(easier)**하게 상호작용할 수 있는 구조입니다.

> And so I'll, I'll highlight the, the memory one first, just because like, I think there are, there are a few memory startups, but actually you don't need them if you just use this one.  

그래서 먼저 메모리 서버를 강조하고 싶은데, 몇몇 메모리 스타트업이 있지만 이 서버 하나만 써도 충분합니다.

> It's also like 200 lines of code.  

코드 분량도 **약 200줄**로 매우 간단하고요.

> It's super simple.  

정말 단순합니다.


>[!important]
>주요 내용
>
 **상태 관리 철학**:
>- 미래 AI 애플리케이션은 상태유지(stateful) 방향
>- 운영 복잡도와 균형 필요
>- Streamable HTTP Transport로 단계적 접근
  >    
**세션 관리**:
>- 세션 재개(resume) 기능 지원
>- 수평적 확장성 확보
>- 네트워크 불안정성 대응
  >  
**인증 방향성**:
>- API 키 직접 노출 방지
>- OAuth 기반 인증 지향
>- 원격 서버 환경 대비


---

### stateless 로 전환

> Stateful to stateless servers.  

상태 유지 서버에서 무상태 서버로의 전환.

> You guys picked SSE as your sort of launch protocol and transport.  

여러분은 처음에 **SSE(Server-Sent Events)** 를 프로토콜 겸 전송 수단으로 선택하셨고요.

> And obviously transport is pluggable.  

그리고 전송 방식은 플러그인처럼 교체 가능하다고요.

> The behind the scenes of that, like was it Jared Palmer's tweet that caused it or were you already working on it?  

 그 결정의 배경은 무엇인가요? Jared Palmer의 트윗 때문이었나요, 아니면 이미 준비 중이셨나요?
 
 >we have GitHub discussions going back, like, you know, in public going back months, really talking about this, this dilemma and the trade-offs involved.  
 >-Justin/David-

아니요, 이미 수개월 전부터 GitHub Discussions에서 이 딜레마와 그에 따른 **트레이드오프(trade-offs)**에 대해 활발히 공개 논의를 해 왔습니다.

> You know, we do believe that like.  

저희는 분명히 이렇게 믿고 있습니다.

> The future of AI applications and ecosystem and agents, all of these things I think will be stateful or will be more in the direction of statefulness.  

AI 애플리케이션·에코시스템·에이전트 등 미래의 많은 기술은 **상태 유지(stateful)** 쪽으로 나아갈 것이라고요.

> So we had a lot of.  

그래서 내부적으로도 많은…

> I think honestly, this is one of the most contentious topics we've discussed as like the core MCP team and like gone through multiple iterations on and back and forth.  

솔직히 말해, MCP 핵심 팀 내부에서 **가장 논쟁적(contentious)** 이었던 주제 중 하나로, 여러 차례 반복 논의를 거쳤습니다.

> But ultimately just came back to this conclusion that like if the future looks more stateful, we don't want to move away from that paradigm. Completely.  

하지만 최종 결론은, 미래가 상태 유지 쪽이라면 이 패러다임을 **완전히 포기**하고 싶지 않다는 것이었습니다.

> Now we have to balance that against it's it's been operationally complex or like it's hard to deploy an MCP server if it requires this like long lived persistent connection.  

다만 **운영 난이도(operational complexity)** 를 고려해야 하는데, 장시간 지속 연결(persistent connection)을 요구하는 서버는 배포가 어렵습니다.

> This this is the original like SSE transport design is basically you deploy an MCP server and then a client can come in and connect.  

원래 **SSE 기반 전송 디자인**은 MCP 서버를 배포하면, 클라이언트가 접속(connect)만 하면 되는 방식이었고요.

> And then basically you should remain connected indefinitely, which is that's like a tall order for anyone operating at scale.  

클라이언트는 **무한 연결(indefinite connection)** 상태를 유지해야 하는데, 대규모 운영 환경에서는 매우 부담스럽습니다.

> It's just like not a deployment or operational model you really want to support it.  

사실상 그 방식을 그대로 지원하기는 어렵죠.

> So we were trying to think, like, how can we balance the belief that statefulness is important with sort of simpler operation and maintenance and stuff like that?  

그래서 “상태 유지의 중요성”과 “운영·유지보수의 단순성” 사이에서 어떻게 균형을 맞출지 고민했습니다.

> And the news sort of we're calling it the streamable HTTP transport that we came up with still has SSE in there.  

결과적으로 저희는 **Streamable HTTP Transport**라는 방식을 제안했는데, 내부적으로는 여전히 SSE를 활용합니다.

> But it has a more like a gradual approach where like a server could be just plain HTTP, like, you know, have one endpoint that you send HTTP posts to and then, you know, get a result back.  

그러면서도 서버를 **순차적(gradual)** 으로 업그레이드할 수 있도록, 우선은 일반 HTTP POST 한 번으로 결과를 받을 수 있는 엔드포인트를 마련했습니다.

> And then you can like gradually enhance it with like, OK, now I want the results to be streaming or like now I want the server to be able to issue its own requests.  

그다음 단계로 “결과를 스트리밍(streaming)으로 받고 싶다”거나 “서버가 클라이언트 요청 없이도 푸시를 하길 원한다” 같은 기능을 순차적으로 추가할 수 있습니다.

> And as long as the server and client both support the ability to like resume sessions, like, you know, to disconnect and come back later and pick up where you left off, then you get kind of the best of both worlds where it could still be this stateful interaction and stateful server, but allows you to like horizontally scale more easily or like deal with spotty network connections or whatever the case may be.  

또한 서버·클라이언트가 **세션 재개(resume sessions)** 를 지원하면, 연결이 끊겼다가 다시 이어도 이전 상태를 복원할 수 있어, 상태 유지와 확장성·불안정 네트워크 대응을 모두 만족시킬 수 있습니다.

---
### 인증

>And you had, as you mentioned, session ID.  

 세션 ID를 언급하셨고요.

>How do you think about auth going forward?  

 앞으로 인증(auth)은 어떻게 설계하실 계획인가요?

>And that will solve a lot of these issues because you don't really want to have people bring API keys, particularly when you have like, when you think about a world, which, which I truly believe will happen where the majority of servers will be remote servers.  

API 키를 그대로 노출하는 것은 위험하기 때문에, 특히 대부분의 서버가 원격 서비스화될 미래를 대비해 OAuth 기반 인증이 더 안전한 해법입니다.

### 오픈소스 제단관련

이후 오픈소스 관련 논의 와 대화를 하면서 끝을 맻는다.
 
공식 발표일은 2024년 11월 25일  
  
  
MCP?  
  
>"Model Context Protocol, or MCP for short, is basically something we've designed to help AI applications extend themselves or integrate with an ecosystem of plugins, basically. The terminology is a bit different. We use this client-server terminology, and we can talk about why that is and where that came from. But at the end of the day, it really is that. It's like extending and enhancing the functionality of AI application."  
  
>"Another version that we've used and gotten to like is like MCP is kind of like the USB-C port of AI applications and that it's meant to be this universal connector to a whole ecosystem of things."  
  


  
  
### 출처  
---  
  
[https://www.latent.space/p/mcp](https://www.latent.space/p/mcp)
