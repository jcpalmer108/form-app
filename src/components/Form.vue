<template>
    <div class="container">
      <progress max=”100” value=”0”></progress>

      <b-form @submit="onSubmit" @reset="onReset">
        <b-form-group
        id="input-email"
        label="Email address:"
        label-for="email"
        description="Enter a valid email address"
      >
        <b-form-input
          id="email"
          v-model="form.emailAddress"
          type="email"
          required
          placeholder="Enter email"
        ></b-form-input>
      </b-form-group>
      <b-form-file
        id="email"
        v-model="form.uploadedFile"
        :state="Boolean(form.uploadedFile)"
        placeholder="Choose a file or drop it here..."
        drop-placeholder="Drop file here..."
      ></b-form-file>
      <div class="mt-3">Selected file: {{ form.uploadedFile ? form.uploadedFile.name : '' }}</div>
      <div class="buttons">
        <b-button type="submit" variant="primary">Submit</b-button>
        <b-button type="reset" variant="danger">Reset</b-button>
      </div>
    </b-form>
  </div>
</template>

<script>
import AWS from 'aws-sdk'
import $ from 'jquery'
import { BUCKET_NAME, BUCKET_REGION, IDENTITY_POOL_ID } from '../awsVariables'

/*
    upload files in s3: https://medium.com/@shresthshruti09/uploading-files-in-aws-s3-bucket-through-javascript-sdk-with-progress-bar-d2a4b3ee77b5
      https://github.com/shresths/upload-files-aws-s3-bucket/blob/master/aws-s3-bucket-upload.html
    build app with Vue and Asp.net core: https://developer.okta.com/blog/2018/08/27/build-crud-app-vuejs-netcore
*/

// TO DO: Link email address and uploaded files to data
// TO DO: Split CSS, HTML, and Javascript into different files to make smaller

export default {
  name: 'Form',
  data () {
    return {
      bucketName: BUCKET_NAME,
      bucketRegion: BUCKET_REGION,
      IdentityPoolId: IDENTITY_POOL_ID,
      s3: undefined,
      form: {
        emailAddress: 'bbb@ggg.com',
        uploadedFile: []
      }
    }
  },
  methods: {
    onSubmit: function () {
      this.s3upload()
      this.sendEmail()
    },
    onReset: function () {
      this.form = {
        emailAddress: '',
        uploadedFile: []
      }
    },
    configureAws: function () {
      try {
        AWS.config.update({
          region: this.bucketRegion,
          credentials: new AWS.CognitoIdentityCredentials({
            IdentityPoolId: this.IdentityPoolId
          })
        })
      } catch (err) {
        console.log('ERROR: Unable to configure AWS')
        console.log(err)
      }
    },
    configureS3: function () {
      try {
        this.s3 = new AWS.S3({
          apiVersion: '2006-03-01',
          params: {Bucket: this.bucketName}
        })
      } catch (err) {
        console.log('ERROR: Unable to load S3')
        console.log(err)
      }
    },
    s3upload: function () {
      try {
        if (this.form.uploadedFile) {
          const params = {
            Bucket: this.bucketName,
            Key: this.form.uploadedFile.name,
            Body: this.form.uploadedFile,
            ACL: 'public-read'
          }

          console.log('file: ', this.form.uploadedFile)
          console.log('params: ', params)

          // Key: filePath,
          // Body: file,
          // ACL: 'public-read'

          // TO DO: Get file uploading to work; access denied error
          if (this.s3) {
            this.s3.putObject(params, function (err, data) {
              if (err) {
                console.log('ERROR: Error uploading')
                console.log(err)
              } else {
                console.log('SUCCESSFUL UPLOAD')
                alert('Successfully Uploaded!')
              }
            }).on('httpUploadProgress', function (progress) {
              var uploaded = parseInt((progress.loaded * 100) / progress.total)
              $('progress').attr('value', uploaded)
            })
          } else {
            console.log('s3 doesnt exist')
          }
        }
      } catch (err) {
        console.log('ERROR: No files uploaded')
        console.log(err)
      }

      console.log('uploaded')
    },
    sendEmail: function () {
      // TO DO: write code that sends email
      console.log('email sent')
    }
  },
  mounted () {
    this.configureAws()
    this.configureS3()
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    .container {
        width: 50%;
    }
    .form-group {
        padding-bottom: 40px;
    }
    .buttons {
      padding-top: 40px;
    }
</style>
