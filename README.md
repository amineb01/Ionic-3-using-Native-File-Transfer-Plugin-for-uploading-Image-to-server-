# Ionic-3-using-Native-File-Transfer-Plugin-for-uploading-Image-to-server-

$ ionic cordova plugin add cordova-plugin-file-transfer
$ npm install --save @ionic-native/file-transfer

$ ionic cordova plugin add cordova-plugin-camera
$ npm install --save @ionic-native/camera


1. in your_component.ts 


                    import { Component, Input, OnInit, OnDestroy } from '@angular/core';
                    import { AccountsProvider } from '../../providers/accounts';
                    import { Subscription } from 'rxjs/Subscription';
                    import { Storage } from '@ionic/storage';
                    import { IonicPage, NavController, NavParams, LoadingController, ToastController } from 'ionic-angular';
                    import { Camera, CameraOptions } from '@ionic-native/camera';
                    import { ModalController } from 'ionic-angular/components/modal/modal-controller';
                    import { ActionSheetController } from 'ionic-angular/components/action-sheet/action-sheet-controller';

                    /**
                     * Generated class for the UserProfileComponent component.
                     *
                     * See https://angular.io/api/core/Component for more info on Angular
                     * Components.
                     */
                    @Component({
                      selector: 'user-profile',
                      templateUrl: 'user-profile.html'
                    })

                    export class UserProfileComponent implements OnInit, OnDestroy {
                      @Input() User: User;
                      $subs: Subscription;
                      pathImage: string = "http://placehold.it/300/300"
                      token: string;
                      constructor(private localStorageService: Storage,
                        private accountService: AccountsProvider
                        , public navCtrl: NavController,

                        private camera: Camera,
                        public loadingCtrl: LoadingController,
                        public toastCtrl: ToastController, private actionSheetCtrl: ActionSheetController, private modalCtrl: ModalController

                      ) {  }



                      ngOnDestroy() {
                        if (this.$subs) this.$subs.unsubscribe()

                      }
                      async   ngOnInit() {
                        let payload = await this.localStorageService.get('payload');
                        this.token = JSON.parse(payload).token
                        this.getImage();
                      }


                      getImage() {
                        this.$subs = this.accountService.getimage(this.User._id, this.token).subscribe(path => { console.log("path   ", path); this.pathImage = path })

                      }


                      presentToast(msg) {
                        let toast = this.toastCtrl.create({
                          message: msg,
                          duration: 3000,
                          position: 'bottom'
                        });

                        toast.onDidDismiss(() => {
                          console.log('Dismissed toast');
                        });

                        toast.present();
                      }


                      presentActionSheet() {
                        let actionSheet = this.actionSheetCtrl.create({
                          title: 'Select Image Source',
                          buttons: [
                            {
                              text: 'Load from Library',
                              handler: () => {
                                this.takePicture(this.camera.PictureSourceType.PHOTOLIBRARY);
                              }
                            },
                            {
                              text: 'Use Camera',
                              handler: () => {
                                this.takePicture(this.camera.PictureSourceType.CAMERA);
                              }
                            },
                            {
                              text: 'Cancel',
                              role: 'cancel'
                            }
                          ]
                        });
                        actionSheet.present();
                      }

                      public takePicture(sourceType) {

                        const options: CameraOptions = {
                          quality: 100,
                          destinationType: this.camera.DestinationType.FILE_URI,
                          sourceType: this.camera.PictureSourceType.PHOTOLIBRARY,
                          encodingType: this.camera.EncodingType.JPEG,
                        }

                        this.camera.getPicture(options).then((imageData) => {
                          this.imageURI = imageData;

                          this.uploadFile();
                        }, (err) => {
                          console.log(err);
                          this.presentToast(err);
                        });

                      }


                      uploadFile() {

                        let loader = this.loadingCtrl.create({
                          content: "Uploading..."
                        });
                        loader.present();
                        console.log("eedsq", this.token)
                        if (this.token) {
                          this.accountService.uploadFile(this.token, this.imageURI).then((data) => {
                            console.log(" Uploaded Successfully", data);
                            this.getImage();


                            loader.dismiss();
                            this.presentToast("Image uploaded successfully");
                          }, (err) => {
                            console.log(err);
                            loader.dismiss();
                            this.presentToast(err);
                          });
                        } else {
                          loader.dismiss();
                          this.presentToast("no token provided");
                        }

                      }


                    }



2. in your_component.html 


        <div class="image">
          <img class="circle-pic" *ngIf="pathImage" [src]="pathImage" />

          <button  class="uploadBu" ion-button icon-only (click)="presentActionSheet()">
            <ion-icon name="md-cloud-download"></ion-icon>
          </button>

      </div>



3. Create your AccountsProvider   services/AccountsProvider.ts


4. in app.module.ts
  
            .... 
            providers: [
              StatusBar,
              SplashScreen,
              AccountsProvider ,
              FileTransfer,
                FileTransferObject,
                File,
                Camera

            ]
          ....






------------------------------------------------------------------------------------------------------------------------
5. in services/AccountsProvider.ts
 
 
           import { Http, Headers, Response, } from '@angular/http';
          import { pipe } from 'rxjs';
          import { FileTransfer, FileUploadOptions, FileTransferObject } from '@ionic-native/file-transfer';

          import { Injectable } from '@angular/core';
          import 'rxjs/add/operator/map';
          import 'rxjs/add/operator/do';
          import { catchError } from 'rxjs/operators/catchError';
          import { Observable } from 'rxjs/Observable';
          import { HttpErrorResponse } from '@angular/common/http/src/response';
          import "rxjs/add/observable/throw";
          import "rxjs/add/operator/catch";
          import { RequestOptions } from '@angular/http/src/base_request_options';

          import { Storage } from '@ionic/storage';

          import * as jwt_decode from "jwt-decode";


          let apiUrl = "<your_server_adress>"

          /*
            Generated class for the AccountsProvider provider.

            See https://angular.io/guide/dependency-injection for more info on providers
            and Angular DI.
          */
          @Injectable()
          export class AccountsProvider {

            constructor(private localStorageService: Storage, public http: Http,  private transfer: FileTransfer,) {
              console.log('Hello AccountsProvider Provider');
            }

            uploadFile(token,imageURI) {
              const fileTransfer: FileTransferObject = this.transfer.create();

              let options: FileUploadOptions = {
                fileKey: 'image',
                httpMethod:'POST',
                fileName:'user_step4#'+ Date.now()+".jpg",
                chunkedMode: false,
                mimeType: "image/jpeg",
                headers: {'x-token':token}
              }

             return fileTransfer.upload(imageURI, apiUrl + 'accounts/uploadsImage', options)

            }


            getimage(id: string, token: string) {
              let headers = new Headers({ 'Content-Type': 'application/json', 'token': token });
              headers.append('x-token', token)

              return this.http.get(apiUrl + "accounts/image/" + id, { headers: headers })
                .map((response: Response) => {

                  let res = response.json().path
                  if (res) {
                    return this. getUrl(res)
                  }else{
                    return "http://placehold.it/300/300"
                  }


                })
                .catch((error: Response) => Observable.throw(error.json()) || "server error");
            }


--------------------------------------------------------------------------------------------------


In server-side you can use your preferred Back-end technology by receiving req.file.<you_file_name> and the token in the header 

And here is an example using Node.js (we use Multer, express.js and mongoose)

        var multer = require('multer')

        var storage = multer.diskStorage({
            destination: function (req, file, cb) {
                cb(null, './uploads/')
            },
            filename:function(req,file,cb)    
            {
               var pathname = file.originalname.split('#');
               var filename = pathname[1];
            if(filename!=undefined)
                    cb(null, filename);            
            }
            });


        var upload = multer({ storage: storage})

        router.post("/uploadsImage",[auth,upload.single('image')] , async (req, res) => {

            if (!req.file) return res.status(400).send({ "message": "image is required" })
            file = req.file.path

            let acc = await Account.findOneAndUpdate({ _id: req.decoded._id}, {

                       image: file

            }, { new: true })

            if (!acc) {
                fs.unlink(file);
                return res.status(400).send({  message: "Utilisateur introuvable !" })

            }

            return res.send({path:file})


        })

        router.get("/image/:id", [auth,objectId], async (req, res) => {
            let account = await Account.findOne({ _id: req.params.id });
            if (!account) return res.status(404).send({ message: "Utilisateur introuvable !" })

            res.send({path:account.image})

        })



