<template>
  <div class="container">
    janus

    <video
      id="video"
      autoPlay
      playsInline
    />
  </div>
</template>

<script>
import Logo from '~/components/NuxtLogo.vue'
import Janus from '~/components/janus'

export default {
  components: {
    Logo
  },
  data() {
    return {
      stream: [],
    };
  },
  mounted() {
    // DOM 조작에 대한 목적
    // 현재 연결 주소 - http

    const serverWebSocket = "ws://13.125.132.255:8188/janus";
      
      console.log(navigator.mediaDevices.getSupportedConstraints());
 
    // const server = 
    const serverHttp = "http://13.125.132.255:8088/janus";6
   
   let pin = null;
   let room = 1234;
    let janus = null;
    let  sfutest = null;
    const opaqueId = "videoroomtest-"+Janus.randomString(12); //opaqueId 값을 통해서 유저 구분
   
       let videoHandlerGame = null //Handle 객체
    
// janus.init - 초기화
// debug: "all" - 모든 디버거 켜기
// callback : 초기화 완료 후 콜백

    Janus.init({
      debug: "all",
      callback: function(){
        console.log("janus 초기화 부분 init ");

         //Create session
        janus = new Janus(
        {
          server: serverWebSocket,
          opaqueId: opaqueId,
          success: function() {
            // 세션 생성, 사용준비 완료
            console.log("여기는 janus 객체 부분 new Janus");
            console.log("Janus 서버 " + janus.getServer());                                                                                                                                  
            console.log("Janus 연결 유무 " + janus.isConnected());
            console.log("Janus 세션 " + janus.getSessionId());

            // 세션을 플러그인에 연결하여 핸들을 생성, peerconnction이 보내고 받는 미디어를  조작가능해짐
            janus.attach( 
              { 
                plugin: "janus.plugin.videoroom",
                opaqueId : opaqueId,
                success: function(pluginHandle) {
                  // Handle객체는 success시 리턴 값으로 들어옴.
                  videoHandlerGame = pluginHandle;
              
                  // 연결된 플러그인 확인, id
                  Janus.log("Plugin attached! (" + videoHandlerGame.getPlugin() + ", id=" + videoHandlerGame.getId() + ")");
									Janus.log("  -- This is a publisher/manager");

                  // 1) 로직 시작
                  // ( 플러그인에 메시지를 보내고 , 즉시 플러그인과 peerconnection협상 (createOffer 뒤에 a 메세지), 어
                  // 떤 일이 일어나기를 기다림.)
                        // Plugin 연결! 'pluginHandle'은 우리 핸들입니다

                      var register = {
                          request: "create",
                          room: room,
                          ptype: "publisher",
                      };
                  
                videoHandlerGame.send({ message: register });
                Janus.log("room 생성.");
                        
                }, 
                error : function(cause) { 
                        // 플러그인에 연결할 수 없습니다 . 
                }, 
                consentDialog: function(on) { 
                  //getUserMedia true 및 완료 된 수에 트리거됨.
                  // 즉, 엑세스 요청 수락
                        // 예, on=true(getUserMedia 수신)인 경우 화면을 어둡게 하고, 그렇지 않으면 복원합니다 . 
                }, 
                mediaState:function(){
                  // Media 정보를 받기 시작하거나 중지할 때 호출된다. (mediaState, type=audio, on=true ⇒ Janus가 Audio Stream을 받기 시작했다.) (mediaState, type=video, on=false ⇒ Janus가 Video Stream을 받지 않기 시작했다.) 
                  // Janus가 Media 정보를 받기 시작하거나 문제가 생겼을 때 핸들링하기 좋다.
                },
                onmessage: function(msg , jsep) {
                  //success 에서 send를 사용하고 이 콜백을 통해 상호작용이 가능해집니다.
                        // 플러그인에서 메시지/이벤트(msg)를 받았습니다. 
                        // jsep이 null이 아니면 WebRTC 협상이 포함됩니다. 

                        Janus.debug(" ::: Got a message (publisher) :::", msg);
                        let event = msg["videoroom"];
                        Janus.debug("Event: " + event);

                        if (event === "event") {
                            let list = msg["publishers"]; //참여자

                            Janus.log(
                              "Got a list of available publishers/feeds:",
                              list
                            );
                          }
                }, 
                onlocaltrack: function(track, added) { 
                        // 표시할 로컬 트랙이 방금 추가되었습니다( getUserMedia 작동!) 또는 제거 
                }, 
                onremotetrack: function(track, mid, added) { 
                        // 특정 mid가 있는 원격 트랙(작동 PeerConnection!)이 방금 추가 또는 제거되었습니다 
                }, 
                oncleanup: function() { 
                        // PeerConnection 플러그인을 닫은 상태에서 UI 청소
                        // 플러그인 핸들은 여전히 ​​유효하므로 새 핸들을 생성할 수 있습니다. 
                }, 
                detached: function() { 
                        // 플러그인과의 연결이 닫히고 기능이 제거됩니다. 
                        // 플러그인 핸들은 더 이상 유효하지 않습니다 . 
                },onremotestream: function (stream) {
      
      },
                
              });
          },
          error: function(cause) {
            console.log("여기는 janus 객체 부분");
            // 세션 생성 실패
                  console.log("세션 생성 실패 ");

          },
          destroyed: function() {
            console.log("여기는 janus 객체 부분");
                  // 세션 파괴, 더이상 사용되지 않음
                  console.log("세션 파괴");
          }
        });
        
      }}) 
  }
}
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}

#video{
border: solid 3px red;
}

</style>
