
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
>자동선택, 설치 확인하기.  
  
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
  
---  
### 도구 사용 개수와 제어 권한 관리  
  
>Anyway, so the first one is, it's just a simple, how many MCP things can one implementation support, you know, so this is the, the, the sort of wide versus deep question.  
>-swyx-  
  
좋습니다. 좀 더 구체적인 질문으로 넘어가 보죠. 첫 번째는 단순합니다. “하나의 구현에서 얼마나 많은 MCP 서버(또는 도구·리소스·프롬프트 등)를 지원할 수 있느냐?”라는 폭(많이) 대 깊이(적게) 문제입니다.  
  
>  
  
  
공식 발표일은 2024년 11월 25일  
  
  
MCP?  
  
>"Model Context Protocol, or MCP for short, is basically something we've designed to help AI applications extend themselves or integrate with an ecosystem of plugins, basically. The terminology is a bit different. We use this client-server terminology, and we can talk about why that is and where that came from. But at the end of the day, it really is that. It's like extending and enhancing the functionality of AI application."  
  
>"Another version that we've used and gotten to like is like MCP is kind of like the USB-C port of AI applications and that it's meant to be this universal connector to a whole ecosystem of things."  
  
  
  
  
### 출처  
---  
  
[https://www.latent.space/p/mcp](https://www.latent.space/p/mcp)
