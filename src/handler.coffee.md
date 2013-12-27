IP Access Protocol Handler
==========================

We do not support the OpenBSC extension, since it is obsolete.

    module.exports = class ipAccessProtocolHandler

      constructor: (@c) ->
        @init()

        @c.on 'ping', =>
          @c.send 'pong'

        @c.on 'pong', =>
          console.log 'Received pong'
          # FIXME: disarm timer PONG

        @c.on 'id_get', =>
          @c.send 'id_resp', Buffer.concat [
            @tag 'unit_name', 'SIP MSC'
            @tag 'unit_id', '1234'
          ]
          # FIXME arm timer ID_ACK

        @c.on 'id_ack', =>
          # FIXME disarm timer ID_ACK

        @c.on 'id_resp', (params) =>
          response = @parse_idtags params
          if response?
            @c.send 'id_ack'
            # disarm timer ID_RESP

        ###
          gather_id =
            if not @id_get_sent?
              @c.send 'id_get', Buffer.concat [
                @tag 'unit_name'
                @tag 'unit_id'
                @tag 'ip_address'
              ]
              @id_get_send = true

          setTimeout gather_id, 500
        ###

### ID tags parsing and building

      tag_id_to_name:
        0x00: 'serial_number'
        0x01: 'unit_name'
        0x02: 'location_1'
        0x03: 'location_2'
        0x04: 'equipment_version'
        0x05: 'software_version'
        0x06: 'ip_address'
        0x07: 'mac_address'
        0x08: 'unit_id'

      init: ->
        @tag_name_to_id = {}
        for id, name of @tag_id_to_name
          @tag_name_to_id[name] = id

Tag Parser
----------

      parse_idtags: (buf) ->
        len = buf.length
        i = 0
        response = {}
        while i < len
          tag_len = buf.readUInt8 i++
          tag_id  = buf.readUInt8 i++

          if i+tag_len > len
            console.error "The tag does not fit: #{tag_len}"
            return null

          tag_name = @tag_id_to_name[tag_id]
          unless tag_name?
            console.error "Unknwon tag id #{tag_id}"
            return null

          response[tag_name] = (buf.slice i, i+tag_len).toString()
          i += tag_len

        return response

Tag Builder
-----------

If a value is provided, build a segment that represents a response for that name and value.
If no value is provided, build a segment that represents a request for that name.

      tag: (name,value) ->
        tag_id = @tag_name_to_id[name]
        if value?
          value = new Buffer value
          tag_len = value.length
          response = new Buffer value.length + 2
          response.writeUInt8 tag_len, 0
          response.writeUInt8 tag_id, 1
          value.copy response, 2
          return response
        else
          response = new Buffer 2
          response.writeUInt8 1, 0
          response.writeUInt8 tag_id, 1
          return response
