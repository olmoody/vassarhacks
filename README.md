# VassarHacks-Project

//
//  ContentView.swift
//  VassarHacks App
//
//  Created by Curtis Fedorko on 5/1/21.
//

import SwiftUI

import SwiftUI
import Foundation
import Photos

struct User: Decodable {
    //var id: ObjectIdentifier
    
    let id = UUID()
    var actuals = Actuals(vaccinationsCompleted: 0)
    
    let population: Int
    let country : String

}
class apiCall {
    func getUsers(completion:@escaping (Newss) -> ()) {
        guard let url = URL(string: "https://vaccovid-coronavirus-vaccine-and-treatment-tracker.p.rapidapi.com/api/news/get-vaccine-news/0") else { return }
        var request = URLRequest(url: url)
        request.httpMethod = "GET"
        // Set HTTP Request Header
        request.setValue("vaccovid-coronavirus-vaccine-and-treatment-tracker.p.rapidapi.com", forHTTPHeaderField: "x-rapidapi-host")
        request.setValue("6944bf73bcmsh49a3a4b0a39ed55p1f9f96jsna86e2030fe49", forHTTPHeaderField: "x-rapidapi-key")
        URLSession.shared.dataTask(with: request) { (data, _, _) in
            let users = try! JSONDecoder().decode(Newss.self, from: data!)
            print(users)
            
            DispatchQueue.main.async {
                completion(users)
            }
        }
        .resume()
    }
}
struct Newss: Decodable {
    var news : [NewsItem]
}
struct NewsItem: Codable,Identifiable {
    let id = UUID()
    var title : String
    var link : String
}
struct Actuals: Decodable {
    let vaccinationsCompleted: Int
}

struct ContentView: View {
    var body: some View {
        NavigationView{
            VStack{
                Text("VidView").font(.system(size: 35, weight: .bold, design: .default )).padding()
                Image("Image").padding()
            NavigationLink(destination: ContentViewMenu()) {
                Text("Menu").font(.system(size: 35, weight: .bold, design: .default ))
            }
            }
        }.navigationBarTitle("Home")
    }}


struct ContentViewy: View {
    @State private var image: Image?
    @State private var filterIntensity = 0.5
    @State private var showUploadVax = false
    @State private var inputImage: UIImage?
    var body: some View {
        NavigationView{
            VStack{
                ZStack{
                    Rectangle()
                        .fill(Color.secondary)
                        if image != nil{
                            image?
                                .resizable()
                                .scaledToFit()
                        }
                        else { Text("Tap to Select a Photo")
                            .foregroundColor(.white)
                            .font(.headline)
                        }
                    
                    }
                .onTapGesture {
                    self.showUploadVax = true
                }
                .padding(.vertical)
                    }
             
                }
            .sheet(isPresented: $showUploadVax, onDismiss: loadImage) {
                UploadVax(image: self.$inputImage)
      
            }
            }
        
    func loadImage() {
        guard let inputImage = inputImage else {return}
        image = Image(uiImage: inputImage)
    }
}
    
    

struct ContentViewMenu: View {
    @State private var image: Image?
    @State private var filterIntensity = 0.5
    @State private var showUploadVax = false
    var body: some View {
        NavigationView{
            List{
                
                NavigationLink(destination: StatsState()) {
                    Text("Stats")
                }.listRowBackground(Color.green)
                NavigationLink(destination: News()) {
                    Text("News")
                }.listRowBackground(Color.green)
               
                NavigationLink(destination: ContentViewy(
                )){
                    Text("Your Vaccine Report")}.listRowBackground(Color.green)
                
                NavigationLink(destination: NavigateFromMenu()){
                    Text("Book A Vaccine")
                }.listRowBackground(Color.green)
            }.navigationTitle("Menu")
        }
    }

                
    
}
struct StatsState:View {
    //@State private var state : String? = nil
    var body: some View {
        NavigationView{
        List{
            NavigationLink(destination: Stats(state:"MA")) {
                Text("Massachusetts")
            }.listRowBackground(Color.green)
            NavigationLink(destination: Stats(state:"NY")){
                Text("New York")
            }.listRowBackground(Color.green)
            NavigationLink(destination: Stats(state:"CA")){
                Text("California")
            }.listRowBackground(Color.green)
            NavigationLink(destination: Stats(state:"ND")){
                Text("North Dakota")
            }.listRowBackground(Color.green)
        
        }
    }.navigationTitle("Pick your state")
    }
}


struct Stats: View {
    @State var dic : User?
    @State var pop : Double? = 0.0
    //let a = apiCall(state:"MA");
    var state = "MA"
    //let dic = NSDictionary()
    var body: some View {
        NavigationView{
            ZStack{
                Color(UIColor.lightGray).edgesIgnoringSafeArea(.all)
                Text("Percent of people vaccinated in \(state):  \(String(self.pop ?? 0))").font(.system(size: 35, weight: .bold, design: .default ))
//                            Text(String(user.population))
//                                .font(.subheadline)
                     
            }.onAppear {
                let urlstring = "https://api.covidactnow.org/v2/state/\(state).json?apiKey=837f0235c0024bc3995d4323c5f215b5"
                guard let url = URL(string: urlstring) else { return }
                

                URLSession.shared.dataTask(with: url){ (data, _, _) in
                      guard let data = data else { return }
                      let movies = try! JSONDecoder().decode(User.self, from: data)
                      DispatchQueue.main.async {
                        self.pop = Double(movies.actuals.vaccinationsCompleted)/Double(movies.population)
                      }
                    }.resume()
        
    
            }
            
        }.navigationTitle("Stats")
    
}
}

struct News: View {
    init() {
        UITableView.appearance().backgroundColor = .lightGray
        }
    @State var dic : String?
    @State var pop : Newss?
    //let a = apiCall(state:"MA");
    var state = "MA"
    //let dic = NSDictionary()
    var body: some View {
        NavigationView{
            List(pop?.news ?? []) {
                user in
//
                Link(user.title, destination: URL(string: user.link)!).font(.system(size: 15, weight: .bold, design: .default )).foregroundColor(Color.green)
                             //  .font(.headline)
//                            Text(String(user.population))
//                                .font(.subheadline)
                     
            }.onAppear {
                
                apiCall().getUsers {(users) in
                    self.pop = users
                    
                }
            
            
        }
    
}.navigationTitle("Latest News")
}
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}


struct NavigateFromMenu: View {
    @Environment(\.openURL) var openURL
    @State private var navigateTo = ""
    @State private var isActive = false
    var body: some View {
        NavigationView {
            Menu {
                Button("AL") {
                    openURL(URL(string: "https://www.alabamapublichealth.gov/covid19vaccine/index.html")!)
                    self.isActive = true
                }
                Button("AK") {
                    openURL(URL(string: "http://dhss.alaska.gov/dph/epi/id/pages/COVID-19/vaccine.aspx")!)
                    self.isActive = true
                }
                Button("CT") {
                    openURL(URL(string: "https://portal.ct.gov/Vaccine-Portal")!)
                    self.isActive = true
                }
                Button("WA") {
                    openURL(URL(string: "https://www.doh.wa.gov/Emergencies/COVID19/vaccine")!)
                    self.isActive = true
                }
                Button("CA") {
                    openURL(URL(string: "https://myturn.ca.gov/")!)
                    self.isActive = true
                }
                Button("NY") {
                    openURL(URL(string: "https://am-i-eligible.covid19vaccine.health.ny.gov/")!)
                    self.isActive = true
                }
                Button("DE") {
                    openURL(URL(string: "https://coronavirus.delaware.gov/vaccine/")!)
                    self.isActive = true
                }
                Button("ME") {
                    openURL(URL(string: "https://www.maine.gov/dhhs/mecdc/infectious-disease/immunization/covid-19-providers/index.shtml")!)
                    self.isActive = true
                }
            } label: {
                Text("Select Your State")
            }
            .background(
                NavigationLink(destination: Text(self.navigateTo), isActive: $isActive) {
                    EmptyView()
                })
        }
    }
}
    
struct UploadVax: UIViewControllerRepresentable {
    class Coordinator: NSObject, UINavigationControllerDelegate, UIImagePickerControllerDelegate {
        let parent: UploadVax

        init(_ parent: UploadVax) {
            self.parent = parent
        }
    
        func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any]) {
            if let uiImage = info[.originalImage] as? UIImage{
                parent.image = uiImage
            }
            parent.presentationMode.wrappedValue.dismiss()
        }
    }
 
    @Environment(\.presentationMode) var presentationMode
        @Binding var image: UIImage?
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }

        func makeUIViewController(context: UIViewControllerRepresentableContext<UploadVax>) -> UIImagePickerController {
            let picker = UIImagePickerController()
            picker.delegate = context.coordinator
            return picker
        }

        func updateUIViewController(_ uiViewController: UIImagePickerController, context: UIViewControllerRepresentableContext<UploadVax>) {

        }
    }
func makeUIViewController(context: UIViewControllerRepresentableContext<UploadVax>) -> UIImagePickerController {
    let picker = UIImagePickerController()
    picker.delegate = context.coordinator
    return picker
}

func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey: Any]) {


}
