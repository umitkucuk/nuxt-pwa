<template>
  <div>
    <div v-if="channelId" class="qr-scanner">
      <qrcode-stream @decode="onDecode"></qrcode-stream>
    </div>
    <div v-else class="channel">
      <a-tooltip
        placement="left"
        title="Buradan kanaldan ayrılabilirsin"
        :visible="!selectedImage"
      >
        <a-button
          class="disconnect"
          type="danger"
          shape="circle"
          icon="close"
          @click="disconnect"
        />
      </a-tooltip>

      <div class="selected-image">
        <img v-if="selectedImage" :src="selectedImage" />
        <h2 v-else style="text-align: center">
          Kanala bağlandınız.
          <br />Resimlerinizi yükleyebilirsiniz. <br />😎
        </h2>
      </div>

      <div class="channel-footer">
        <a-upload
          name="file"
          class="upload-image"
          accept="image/*"
          :multiple="true"
          :show-upload-list="false"
          @change="handleSelectImage"
        >
          <a-tooltip
            placement="top"
            title="Buradan resim yükleyebilirsin"
            :visible="!selectedImage"
          >
            <a-button>
              <a-icon :type="loading ? 'loading' : 'plus'" />
              {{ loading ? 'Yükleniyor' : 'Resim Yükle' }}
            </a-button>
          </a-tooltip>
        </a-upload>
      </div>
    </div>
  </div>
</template>

<script>
import io from 'socket.io-client'
import { QrcodeStream } from 'vue-qrcode-reader'

function getBase64(img, callback) {
  const reader = new FileReader()
  reader.addEventListener('load', () => callback(reader.result))
  reader.readAsDataURL(img)
}

export default {
  components: {
    QrcodeStream
  },
  data() {
    return {
      socket: io('https://makeimagebigger.herokuapp.com'),
      channelId: null,
      selectedImage: null,
      images: [],
      loading: false,
      sending: false
    }
  },
  methods: {
    onDecode(decodedString) {
      this.channelId = decodedString
      this.joinChannel(decodedString)
    },
    joinChannel(channelId) {
      this.socket.emit('JOIN_CHANNEL', channelId)
      this.socket.on('SEND_IMAGE', (data) => {})
    },
    disconnect() {
      //
    },
    handleSelectImage(info) {
      if (info.file.status === 'uploading') {
        this.loading = true
        return
      }

      if (info.file.status === 'done') {
        this.$message.success('Resim başarıyla yüklendi 🙃')
        getBase64(info.file.originFileObj, (imageUrl) => {
          alert('yüklendi')
          this.selectedImage = imageUrl
          this.loading = false
          this.sendImage()
        })
      }
    },
    sendImage() {
      this.socket.emit(
        'SEND_IMAGE',
        {
          channel_id: this.channelId,
          image: this.selectedImage
        },
        () => {
          // this.$message.success('Resim başarıyla gönderildi 🙃')
        }
      )
    }
  }
}
</script>

<style lang="scss">
.channel {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100vh;

  .selected-image {
    display: flex;
    flex-grow: 1;
    justify-content: center;
    align-items: center;

    img {
      max-width: 100%;
    }
  }

  .channel-footer {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    height: 80px;
    background-color: #fff;
    box-shadow: 0 -1px 10px 1px #edf2f7;
  }

  .disconnect {
    position: absolute;
    top: 20px;
    right: 20px;
  }
}

.upload-image button {
  height: 52px;
  font-size: 22px;
}
</style>
