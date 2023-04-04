# NewScroll
BBC뉴스 api를 받아 리스트를 생성하고 리스트아이템을 클릭하면 본문으로 이동

- 리스트 화면
<img src = "https://user-images.githubusercontent.com/119280160/229792508-ce39647d-df24-40ff-9409-9aa7bc0b24ce.png" width = "200" height = "400"/>

- 기사 본문
<img src = "https://user-images.githubusercontent.com/119280160/229792554-05705513-92cb-4e7a-9c87-6aaa4b42a46f.png" width = "200" height = "400"/>

- 코드

 '''c
 
import SwiftUI
import SwiftyJSON
import SDWebImageSwiftUI
import WebKit

struct ContentView: View {
    @ObservedObject var list = getData()
    var body: some View {
        NavigationStack{
            List(list.datas){
            i in
                NavigationLink(destination: webView(url: i.url)
                    .navigationBarTitle("",displayMode: .inline)){
                    HStack(spacing:15){
                        VStack(alignment: .leading, spacing: 10){
                            Text(i.title).fontWeight(.heavy)
                            Text(i.desc).lineLimit(2).fontWeight(.light)
                        }
                        WebImage(url: URL(string: i.image)!,options: .highPriority,context: nil)
                            .resizable()
                            .frame(width: 130, height: 130)
                            .cornerRadius(20)
                    }.padding(.vertical, 15)
                }
            }.navigationTitle("NewScroll")
        }
            }
    
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}

struct dataType : Identifiable{
    var id: String
    var title: String
    var desc: String
    var url: String
    var image: String
}

class getData: ObservableObject{
    @Published var datas = [dataType]()
    
    init(){
        let source = "https://newsapi.org/v2/top-headlines?country=us&apiKey=bb3b0bb92904467f80bf4fa988a7c4ae"
        
        let url = URL(string: source)!
        
        let session = URLSession(configuration: .default)
        
        session.dataTask(with: url){
            (data, _, err) in
            if err != nil{
                print((err?.localizedDescription)!)
                return
            }
            
            
            let json = try! JSON(data: data!)
            
            for i in json["articles"]{
                let title = i.1["title"].stringValue
                let desc = i.1["description" ].stringValue
                let url = i.1["url"].stringValue
                let image = i.1["urlToImage"].stringValue
                let id = i.1["publishedAt"].stringValue
                DispatchQueue.main.async {
                    self.datas.append(dataType(id: id, title: title, desc: desc, url: url, image: image))
                }
               
            }
            
        }.resume()
    }
}

//Webkit import
struct webView : UIViewRepresentable{
    var url: String
    func makeUIView(context: UIViewRepresentableContext<webView>) -> WKWebView {
        let view = WKWebView()
        view.load(URLRequest(url: URL(string: url)!))
        return view
    }
    
    func updateUIView(_ uiView: WKWebView, context: UIViewRepresentableContext<webView>) {
        
    }
}

 '''
