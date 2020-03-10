<template>
  <div>
    <div v-if="channelId" class="qr-scanner">
      <qrcode-stream @decode="onDecode"></qrcode-stream>
    </div>
    <div v-else class="channel">
      <a-tooltip
        placement="left"
        title="Buradan kanaldan ayrılabilirsin"
        :visible="!loading && !selectedImage"
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
        <div class="upload-button">
          <a-tooltip
            placement="top"
            title="Buradan resim yükleyebilirsin"
            :visible="!loading && !selectedImage"
          >
            <a-button>
              <a-icon :type="loading ? 'loading' : 'plus'" />
              {{ loading ? 'Yükleniyor' : 'Resim Yükle' }}
            </a-button>
          </a-tooltip>
          <input type="file" accept="image/*" @change="handleSelectImage" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import io from 'socket.io-client'
import { QrcodeStream } from 'vue-qrcode-reader'

function getBase64(event, callback) {
  const file = event.target.files[0]
  const reader = new FileReader()
  reader.onload = (e) => callback(e.target.result)
  reader.readAsDataURL(file)
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
      this.socket.emit('LEAVE_CHANNEL', this.channelId, () => {
        this.$router.push('/')
      })
    },
    handleSelectImage(event) {
      this.loading = true
      getBase64(event, (imageUrl) => {
        this.$message.success('Resim başarıyla yüklendi 🙃')
        this.selectedImage = imageUrl
        this.loading = false
        this.sendImage()
      })
    },
    handleInputChange(event) {},
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

    .upload-button {
      position: relative;
      overflow: hidden;
      display: inline-block;

      input[type='file'] {
        position: absolute;
        top: 0;
        bottom: 0;
        left: 0;
        right: 0;
        opacity: 0;
        cursor: pointer;
        width: 100%;
      }

      button {
        height: 52px;
        font-size: 22px;
      }
    }
  }

  .disconnect {
    position: absolute;
    top: 20px;
    right: 20px;
  }
}
</style>
