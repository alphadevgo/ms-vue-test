<template>
	<div class="livePage">
		<div class="face-panel" v-if="sysInfo.supportface == '1'" v-show="isFaceMode">
			<ul v-show="!faceMSCEnable"
				class="infinite-list"
				v-infinite-scroll="load"
				style="overflow:auto"
			>
				<li
					v-for="(item, i) in faceTable"
					class="infinite-list-item"
					:key="i"
                    v-show="item.show"
				>
					<img
						:src="item.src"
                        @click="faceAttribute(i)"
					/>
					<div class="face-info">
						<i :class="item.genderClass" :title="item.genderTitle"></i>
                        <i :class="item.ageClass" :title="item.ageTitle"></i>
						<i :class="item.glassClass" :title="item.glassTitle"></i>
						<i :class="item.maskClass" :title="item.maskTitle"></i>
						<label class="face-time" v-text="item.time"></label>
					</div>
				</li>
			</ul>
            <div id="faceMSCTip" v-show="faceMSCEnable">{{ $t('mscLiveVideoTips') }}</div>
		</div>
		<div class="videoPanel">
			<div class="topBar">
				<div class="leftBar" v-show="liveVideoPri">
					<live-select
                        skinClass="stream-select"
						:options="streamOptions"
						:labelInit="selStreamName"
						@valChange="change_stream"
						:noPad="true"
                        v-show="showStream"
                        v-if="isLoaded"
					></live-select>
                    <live-select
                        skinClass="protocol-select"
						:options="protocolOptions"
						:labelInit="selProtocolName"
						@valChange="changeProtocol"
						:noPad="true"
                        v-if="isLoaded && this.isIE"
					></live-select>
                    <live-select
                        skinClass="smooth-select"
						:options="smoothOptions"
						:labelInit="smoothOptions[smoothSel].label"
						@valChange="changeSmooth"
						:noPad="true"
                        v-if="isLoaded && this.isIE"
					></live-select>
					<live-select
                        skinClass="overlay-select"
						:options="overlayOptions"
						:labelInit="overlayOptionsLable"
						@valChange="change_overlay"
						:noPad="true"
                        v-show="showOverlay"
                        v-if="isLoaded"
					></live-select>
                    <live-select
                        skinClass="lpr-select"
						:options="lprOptions"
						:labelInit="lprOptions[lprSel].label"
						@valChange="change_lpr_lays"
						:noPad="true"
                        v-show="isLprMode"
                        v-if="isLoaded && isLpr && lprStatus"
					></live-select>
                    <live-select
                        skinClass="face-select"
						:options="faceOptions"
						:labelInit="faceOptions[faceSel].label"
						@valChange="change_face_lays"
						:noPad="true"
                        v-show="isFaceMode"
                        v-if="isLoaded"
					></live-select>
				</div>
				<div class="rightImg">
					<i class="icon-alarm" v-show="showAlarm" :title="$t('vca')"></i>
					<i class="icon-tabs-record" v-show="showRecordAlarm" :title="$t('recording')"></i>
					<i class="icon-advanced-schedule-mode" v-show="showScheAlarm" :title="$t('tasking')"></i>
					<i class="icon-motion" v-show="showMotion" :title="$t('motionDetection')"></i>
					<i class="icon-newalarm" v-show="showNewAlarm" :title="$t('alarm')"></i>
					<i class="icon-radar" v-show="showRadarAlarm"></i>
					<i class="icon-iotalarm" v-show="showIotAlarm" :title="IOT"></i>
					<i class="icon-people-count" v-show="showCountAlarm"></i>
					<i class="icon-lpr-alarm-prohibit lpr-black" v-show="showLprAlarm == 1"
                        :title="$t('lprBlack')"></i>
					<i class="icon-lpr-alarm lpr-white" v-show="showLprAlarm == 2"
                        :title="$t('lprWhite')"></i>
					<i class="icon-lpr-alarm lpr-visitor" v-show="showLprAlarm == 3"
                        :title="$t('lprVisitor')"></i>
				</div>
			</div>
            <div class="canvasBox">
                <div ref="canvas" class="video-box" :class="{'lpr-videoroom': isLprMode,'origin': scaleSel=='100%'}">
                    <div
                        id="VideoAxRoom"
                        name="VideoAxRoom"
                        allowfullscreen="true"
                        v-show="showCanvas"
                        ref="video"
                    ></div>
                    <div id="MaxConnTips" class="videoTips" v-show="show_max_conn_tip"></div>
                    <!-- <div class="lpr-info" v-if="isLpr" v-show="isLprMode">
                        <img id="plate_bigimg"  />
                        <div class="lpr-info-txt">
                            <div class="lpr-plate-box">
                                <span>{{$t("lprresult")}}</span>
                                <span class="lpr-plate" :style="{direction: isIran ? 'rtl' : 'ltr'}"
                                >{{lprInfo.plate}}</span>
                            </div>
                            <div class="lpr-plate-info">
                                <span>{{$t("plateType")}}:<label id='lprtype'>{{lprInfo.type||'-'}}</label></span>
                                <span>{{$t("plateColor")}}:<label id='lprcolor'>{{lprInfo.color||'-'}}</label></span>
                                <span>{{$t("vehicleType")}}:<label id='lprvtype'>{{lprInfo.vType||'-'}}</label></span>
                                <span>{{$t("vehicleColor")}}:<label id='lprvcolor'>{{lprInfo.vColor||'-'}}</label></span>
                                <span>{{$t("speed")}}:<label id='lprspeed'>{{lprInfo.speed||'-'}}</label></span>
                                <span>{{$t("direction")}}:<label id='lprdir'>{{lprInfo.direction||'-'}}</label></span>
                            </div>
                        </div>
                    </div> -->
                    <div class="lpr-info" v-if="isLpr" v-show="isLprMode">
                        <img id="plate_bigimg"  />
                        <div class="lpr-info-txt">
                            <div class="lpr-plate-box">
                                <span>{{$t("lprresult")}}</span>
                                <span class="lpr-plate" id='lprplate' :style="{direction: isIran ? 'rtl' : 'ltr'}"
                                ></span>
                            </div>
                            <div class="lpr-plate-info">
                                <span>{{$t("plateType")}}:<label id='lprtype'>-</label></span>
                                <span>{{$t("plateColor")}}:<label id='lprcolor'>-</label></span>
                                <span>{{$t("vehicleType")}}:<label id='lprvtype'>-</label></span>
                                <span>{{$t("vehicleColor")}}:<label id='lprvcolor'>-</label></span>
                                <span>{{$t("speed")}}:<label id='lprspeed'>-</label></span>
                                <span>{{$t("direction")}}:<label id='lprdir'>-</label></span>
                            </div>
                        </div>
                    </div>
                </div>
                <!-- <div class="lpr-list" v-if="isLpr" v-show="isLprMode" >
                    <el-table :data="lprData" row-key="index" lazy>
                        <el-table-column :label="$t('numOf')" prop="index"></el-table-column>
                        <el-table-column :label="$t('licensePlate')" prop="plate"></el-table-column>
                        <el-table-column :label="$t('snapshot')">
                            <template slot-scope="scope">
                                <img :id="`plate_img${scope.$index}`" :src="scope.row.imgSrc" 
                                    @load="plate_img_load(scope.$index, scope.row)"/>
                            </template>
                        </el-table-column>
                        <el-table-column :label="$t('plateType')" prop="type"></el-table-column>
                        <el-table-column :label="$t('plateColor')" prop="color"></el-table-column>
                        <el-table-column :label="$t('vehicleType')" prop="vType"></el-table-column>
                        <el-table-column :label="$t('vehicleColor')" prop="vColor"></el-table-column>
                        <el-table-column :label="$t('speed')" prop="speed"></el-table-column>
                        <el-table-column :label="$t('direction')" prop="direction"></el-table-column>
                        <el-table-column :label="$t('detectionRegion')" prop="region"></el-table-column>
                        <el-table-column :label="$t('time')" prop="time" show-overflow-tooltip></el-table-column>
                        <el-table-column :label="$t('operation')">
                            <template slot-scope="scope">
                                <el-button
                                    @click="show_lpr_img(scope.$index)"
                                    icon="icon-search"
                                ></el-button>
                                <el-button
                                    icon="icon-list"
                                    @click="show_lpr_list(scope.$index)"
                                    :disabled="scope.row.type != $t('visitor')"
                                ></el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </div> -->
                <div class="lpr-list" v-if="isLpr" v-show="isLprMode">
                    <table>
                        <tr>
                            <th width="5%">{{$t("numOf")}}</th>
                            <th width="13%">{{$t("licensePlate")}}</th>
                            <th width="13%">{{$t("snapshot")}}</th>
                            <th width="8%">{{$t("plateType")}}</th>
                            <th width="8%">{{$t("plateColor")}}</th>
                            <th width="8%">{{$t("vehicleType")}}</th>
                            <th width="8%">{{$t("vehicleColor")}}</th>
                            <th width="5%">{{$t("speed")}}</th>
                            <th width="5%">{{$t("direction")}}</th>
                            <th width="13%">{{$t("detectionRegion")}}</th>
                            <th width="13%">{{$t("time")}}</th>
                            <th width="13%">{{$t("operation")}}</th>
                        </tr>
                    </table>
                    <table id='lpr_list' class="el-table">
                        <tr style="visibility:hidden">
                            <th width="5%">{{$t("numOf")}}</th>
                            <th width="13%">{{$t("licensePlate")}}</th>
                            <th width="13%">{{$t("snapshot")}}</th>
                            <th width="8%">{{$t("plateType")}}</th>
                            <th width="8%">{{$t("plateColor")}}</th>
                            <th width="8%">{{$t("vehicleType")}}</th>
                            <th width="8%">{{$t("vehicleColor")}}</th>
                            <th width="5%">{{$t("speed")}}</th>
                            <th width="5%">{{$t("direction")}}</th>
                            <th width="13%">{{$t("detectionRegion")}}</th>
                            <th width="13%">{{$t("time")}}</th>
                            <th width="13%">{{$t("operation")}}</th>
                        </tr>
                    </table>
                </div>
            </div>
			
			<div class="btnBar">
				<div class="btnLeft" v-if="liveVideoPri">
					<i :class="{ 'icon-play': isPlaying == 0, 'icon-play2': isPlaying == 1,'in-effect':true }"
                        :title="isPlaying ? $t('stop'): $t('play')"
                         @click="click_play"></i>
                    <i class="icon-sound" :class="{'in-effect': playAudio}" @click="click_audio"
                        v-if="showMute" :title="$t('mute')"
                        ></i>
					<i class="icon-cut" @click="click_cut" :title="$t('snapshot')"></i>
					<i class="icon-record" :class="{'in-effect': isRecording}" @click="click_record"
                        :title="isRecording ? $t('stopRecord'): $t('startRecord')"></i>
					<i class="icon-plus" :class="{'in-effect': zoomEnable}" @click="click_zoom" 
                        v-show="showPlus" :title="$t('digitalZoom')"></i>
                    <i class="icon-speak" :class="{'in-effect': playTalk}" @click="click_talk"
                        v-if="showSpeaker" :title="playTalk ? $t('stopTalk'): $t('startTalk')"
                        ></i>
                    <i class="icon-face" :class="{'in-effect': isFaceMode}" @click="changeFaceMode"
                        v-if="sysInfo.supportface=='1' && showFaceBtn" :title="$t('faceDetection')"
                    ></i>
                    <i class="icon-alarm-1 output-button" :class="{'in-effect': visible}"
                            @click="visible = !visible"
                            v-if="manualoutputSupport"
                            :title="$t('manualOutput')">
                    <div id="output-alarm-popover"
                        v-if="visible"
                        title=""
                        @click.stop="visible=true">
                        <div style="margin-bottom: 5px">
                            <span v-if="!showExt" class="output-span">{{ $t('dOutput')}}</span>
                            <span v-if="showExt" class="output-span">{{ $t('dOutput') + '1'}}</span>

                            <el-button
                                type="primary" plain @click="isStart = !isStart; click_output_stus(0);"
                                v-text="isStart ? $t('outClose') : $t('outStart')"
                                >{{ $t("edit") }}</el-button>
                        </div>
                        <div v-if="showExt">
                            <span class="output-span">{{ $t('dOutput') + '2'}}</span>
                            <el-button
                                type="primary" plain @click="isStart1 = !isStart1; click_output_stus(1);"
                                v-text="isStart1 ? $t('outClose') : $t('outStart')"
                                >{{ $t("edit") }}</el-button>
                        </div>
                        <iframe class="bot-iframe"></iframe>
                    </div>
                    </i>
				</div>
				<div class="btnRight" v-if="liveVideoPri">
                    <live-select
						:iconClass="windowIcon"
						@valChange="send_fish_windows"
						:noPad="true"
						ulPosition="bottom"
                        ref="fishwindow"
                        v-if="showFishWindow"
					>
						<template>
							<li class="icon-play2" @mousedown="send_fish_windows(0)"></li>
							<li class="icon-display5" @mousedown="send_fish_windows(1)"></li>
							<li class="icon-fives" @mousedown="send_fish_windows(2)"></li>
						</template>
					</live-select>
					<live-select
						:iconClass="scaleMode"
						:labelInit="scaleSel"
						@valChange="change_scale"
						:noPad="true"
						ulPosition="bottom"
                        ref="scale"
                        :title="$t('windowSize')"
					>
						<template>
							<li class="icon-autosize" @click="change_scale(0)">AUTO</li>
							<li class="icon-a-100" @click="change_scale(1)" v-show="!isLprMode">100%</li>
						</template>
					</live-select>
					<i class="icon-fullscreen" @click="do_full_screen" :title="$t('fullScreen')"></i>
				</div>
			</div>
            <label class="icon-menu-right triggerRight" :class="{rotate:!showRight}" v-if="showTriggleRightBtn" @click="triggleRight">
                <iframe class="bot-iframe"></iframe>
            </label>
		</div>
		<div :class="{ 'leftPanel': true, 'disableClick': isAnony }" v-show="showRight">
			<el-tabs v-model="activeTab" class="tabsBox" type="border-card">
				<el-tab-pane name="fishTab" v-if="showFishPanel || showVcaInstall">
					<span slot="label" class="icon-showfish"></span>
					<div class="fishPanel">
						<div class="dewarpBox" v-if="showFishPanel && showDewarpBox">
							<h3>{{ $t("dewarpingRule") }}</h3>
							<div class="flexBox">
								<i :title="$t('hardwareDewarping')"
                                    class="icon-harddewarp" :class="{'in-effect': dewarpType == 0}"
                                    @mousedown="correction_model_switch(0)"></i>
								<i :title="$t('softwareDewarping')"
                                    class="icon-softdewarp" :class="{'in-effect': dewarpType == 1}"
                                    @mousedown="correction_model_switch(1)"></i>
							</div>
						</div>
						<div class="installBox">
							<h3>{{ $t("installation") }}</h3>
							<div class="flexBox">
                                <i :title="$t('wall')"
                                    class="icon-rightinstall"
                                    :class="{'in-effect': vcaInstall == 0}"
                                    v-if="showVcaInstall"
                                    @mousedown="send_vca_install(0)"></i>
								<i :title="$t('ceiling')"
                                    class="icon-topinstall"
                                    :class="{'in-effect': isFisheye ? curFishInstall == 0 : vcaInstall == 1}"
                                    @mousedown="isFisheye ? send_fish_install(0) : send_vca_install(1)"></i>
								<i :title="$t('wall')"
                                    class="icon-rightinstall"
                                    :class="{'in-effect': curFishInstall == 2}"
                                    v-if="showFishPanel"
                                    @mousedown="send_fish_install(2)"></i>
								<i :title="$t('flat')"
                                    class="icon-botinstall" :class="{'in-effect': curFishInstall == 1}"
                                    v-if="showFishPanel"
                                    @mousedown="send_fish_install(1)"></i>
							</div>
						</div>
						<div class="displayBox" v-if="showFishPanel">
							<h3>{{ $t("display") }}</h3>
							<div class="flexBox">
								<i v-for="item in displayArr" :key="item"
                                    :title="displayTitleArr[item]"
                                    :class="[`icon-display${item + 1}`, curDisplay == item ? 'in-effect' : '']"
                                    @mousedown="send_fish_display(item)"
                                    v-show="item == 7 ? showDisplay1O3R : true"></i>
							</div>
						</div>
						<div class="channelBox" v-if="showFishPanel && fishCorrect == 0" v-show="dewarpType == 0">
							<h3>{{ $t("channelFisheye") }}</h3>
							<ul class="listBox">
								<li v-for="(item, i) in channelPlay" :key="i" :class="{'in-effect': item}"
                                    @click="click_channel(i)">
									<i class="icon-channel" :class="{'in-effect': item}"></i
									>{{ `${$t("channelFisheye")} 0${i + 1}` }}
                                    <i v-if="channelPlay.length == 1" class="icon-toolbarpristream"
                                        :class="{'in-effect': item == 1}" :title="$t('primary')"
                                        @click="set_special_channel_play_state(0)"></i>
                                    <i v-if="channelPlay.length == 1"  class="icon-toolbarsecstream"
                                        :class="{
                                        'in-effect': (item == 2 && fishSubEnable[fishDisplay] == '1') , 
                                        'fish-is-disabled': fishSubEnable[fishDisplay] == '0'}"
                                        :title="$t('secondary')"                                      
                                        @click="set_special_channel_play_state(1)"></i>
								</li>
							</ul>
						</div>
					</div>
				</el-tab-pane>
				<el-tab-pane name="ptzTab" v-if="showPtzPanel">
					<span slot="label" class="icon-showptz-line"></span>
					<div class="ptzPanel">
						<ul :class="{'ptz-ctrl': true,  'disableClick': !isSupportPtzCtrl}">
							<li class="icon-upleft"
                                :title="$t('leftUp')"
                                @mousedown="send_ptz_cmd(8)" @mouseup="send_ptz_stop_cmd" @mouseout="ptz_mouse_out"></li>
							<li class="icon-ptzup"
                                :title="$t('up')"
                                @mousedown="send_ptz_cmd(5)" @mouseup="send_ptz_stop_cmd" @mouseout="ptz_mouse_out"></li>
							<li class="icon-upright"
                                :title="$t('rightUp')"
                                @mousedown="send_ptz_cmd(9)" @mouseup="send_ptz_stop_cmd" @mouseout="ptz_mouse_out"></li>
							<li class="icon-ptzleft"
                                :title="$t('left')"
                                @mousedown="send_ptz_cmd(7)" @mouseup="send_ptz_stop_cmd" @mouseout="ptz_mouse_out"></li>
							<li class="icon-ptzauto" :class="{'in-effect': autoScanStatus}"
                                :title="$t('autoScan')"
                                @mousedown="click_auto_scan"></li>
							<li class="icon-ptzright"
                                :title="$t('right')"
                                @mousedown="send_ptz_cmd(12)" @mouseup="send_ptz_stop_cmd" @mouseout="ptz_mouse_out"></li>
							<li class="icon-downleft"
                                :title="$t('leftDown')"
                                @mousedown="send_ptz_cmd(10)" @mouseup="send_ptz_stop_cmd" @mouseout="ptz_mouse_out"></li>
							<li class="icon-ptzdown"
                                :title="$t('down')"
                                @mousedown="send_ptz_cmd(6)" @mouseup="send_ptz_stop_cmd" @mouseout="ptz_mouse_out"></li>
							<li class="icon-downright"
                                :title="$t('rightDown')"
                                @mousedown="send_ptz_cmd(11)" @mouseup="send_ptz_stop_cmd" @mouseout="ptz_mouse_out"></li>
						</ul>
						<li :class="{'icon-ptzSpeed': true,  'disableClick': !isSupportPtzCtrl}"
                            :title="$t('ptzSpeed')">
							<el-slider :min="1" :max="10" v-model="ptzSpeed" @change="change_ptz_speed" :show-tooltip="true"></el-slider>
						</li>
						<ul class="ptz-ctrl focus-ctrl">
							<li :class="{'icon-zoom1 btn-border': true,
                                'disableClick': (!isSupportPtzCtrl || disableZoom)}" 
                                v-if="isAutoF"
                                :title="$t('zoomOut')"
                                @mousedown="send_ptz_cmd(4)" @mouseup="send_ptz_stop_cmd" @mouseout="ptz_mouse_out"></li>
							<li :class="{'icon-zoom2 btn-border': true,
                                'disableClick': (!isSupportPtzCtrl || disableZoom)}" 
                                 v-if="isAutoF"
                                :title="$t('zoomIn')"
                                @mousedown="send_ptz_cmd(3)" @mouseup="send_ptz_stop_cmd" @mouseout="ptz_mouse_out"></li>
							<li :class="{'icon-focusminusb btn-border': true, 
                                'disableClick': (isFisheye || !isSupportPtzCtrl || disableFocus)}"
                                v-if="isAutoF"
                                :title="$t('focusOut')"
                                @mousedown="send_ptz_cmd(2)" @mouseup="send_ptz_stop_cmd" @mouseout="ptz_mouse_out"></li>
							<li :class="{'icon-focusplusb btn-border': true,
                                'disableClick': (isFisheye || !isSupportPtzCtrl || disableFocus)}"
                                v-if="isAutoF"
                                :title="$t('focusIn')"
                                @mousedown="send_ptz_cmd(1)" @mouseup="send_ptz_stop_cmd" @mouseout="ptz_mouse_out"></li>
                            <li class="icon-irisminusb btn-border" v-if="!noManualIris"
                                :title="$t('irisOut')"
                                @mousedown="send_iris(0)" @mouseup="send_ptz_stop_cmd" @mouseout="ptz_mouse_out"></li>
							<li class="icon-irisplusb btn-border" v-if="!noManualIris"
                                :title="$t('irisIn')"
                                @mousedown="send_iris(1)" @mouseup="send_ptz_stop_cmd" @mouseout="ptz_mouse_out"></li>
							<li class="icon-default" v-if="showLensDiv" @mousedown="send_ptz_cmd(17)"
                                :title="$t('lensInit')"
                                :class="{'disableClick': disableLens}"></li>
							<li class="icon-deffocb" v-if="showLensDiv" @mousedown="send_ptz_cmd(16)"
                                :title="$t('auxilyFocus')"
                                :class="{'disableClick': disableAutoFocus}"></li>
							<li class="icon-autoiris"
                                :class="{'in-effect': irisMode,
                                    'locked-active': irisMode,
                                    'locked-disabled': !irisMode}"
                                v-if="showLensDiv" :title="$t('autoIris')"
                                @click="change_iris_mode"></li>
						</ul>
						<ul :class="{'ptz-feature':true}">
							<li class="icon-light" :class="{'in-effect': lightMode,
                                'disableClick': disablePTZFeature}" @click="change_light"
								v-if="isShowLight"
								:title="lightMode == '1' ? $t('turnOffLight') : $t('light30s')"></li>
							<li class="icon-a-3d" :class="{'in-effect': posEnable,
                                'disableClick': disablePTZFeature || !isSupportPtzCtrl}" @click="change_pos"
                                :title="$t('3dPos')"></li>
							<li class="icon-a-1key" :class="{'in-effect': (patrolMode == 1),
                                'disableClick': disablePTZFeature}" @click="change_patrol"
                                :title="$t('oneTouch')"></li>
							<li class="icon-position" :class="{'in-effect': watchEnable,
                                'disableClick': disablePTZFeature}" @click="change_watch"
                                :title="$t('autoHome')"></li>
							<li :class="{'icon-track': sysInfo.isfisheye == '0',
                                'icon-fisheyetrack': sysInfo.isfisheye == '1',
                                'in-effect': isFisheye ? trackEnable && showTrack && !disableTrack : trackEnable && manualTrack,
                                'disableClick': disablePTZFeature && disableTrack}"
                                :title="sysInfo.isfisheye == '1' ? $t('autoTracking') : $t('manualTracking')"
                                @click="change_track"></li>
                            <li class="icon-defog" :class="{'in-effect': manualDefog,
                                'disableClick': disablePTZFeature}" @click="change_defog"
                                v-if="showDefog" :title="$t('dehumidifying')"></li>
						</ul>
						<el-tabs
							v-model="activePTZ"
							:class="{'ptz-list': true, 'disableClick': isForbidPtzCtrlAndSettings}"
							v-show="setTour == -1"
						>
							<el-tab-pane name="ptzPreset">
								<span slot="label" class="icon-preset" :title="$t('preset')"></span>
								<ul
									class="infinite-list preset-list"
									style="overflow:auto"
								>
									<li
										v-for="(item, i) in presetTable"
										class="infinite-list-item"
										:key="i"
									>
										<label>{{ format_preset_id(i + 1, 3) }}</label>
										<label class="preset-name" :title="item.name" @click="click_preset_label(i)"
                                            v-show="editPreset != i">{{ item.name }}</label>
                                        <el-input v-model="item.name" v-show="editPreset == i" :ref="`preset${i}`"
                                            maxlength="20"
                                            @blur="presetInputBlur(i)"></el-input>
										<span v-show="!displayDisablePtz">
											<i
												class="icon-save" v-show="isSupportPtzSettings && item.quick == 0"
                                                @click="save_preset(i)"
                                                :title="$t('set')"
											></i>
											<el-button 
                                                v-show="isSupportPtzSettings && item.exist == 1 && item.quick == 0"
												@click="del_preset(i)" class="icon-del"
                                                :title="$t('delete')"
											></el-button>
											<el-button v-show="isSupportPtzCtrl && item.exist == 1"
                                                @click="goto_preset(i)"
												class="icon-share"
                                                :title="$t('call')"
											></el-button>
										</span>
									</li>
								</ul>
							</el-tab-pane>
							<el-tab-pane name="ptzTour">
								<span slot="label" class="icon-tour" :title="$t('patrol')"></span>
								<ul
									class="infinite-list tour-list"
									style="overflow:auto"
								>
									<li
										v-for="i in PATROL_NUM"
										class="infinite-list-item"
										:key="i"
									>
										<label>{{ format_preset_id(i, 3) }}</label>
										<label class="tour-name">{{ `${$t('path')} ${i}` }}</label>
										<span v-show="!displayDisablePtz">
                                            <el-button v-show="isSupportPtzCtrl && tourArr[i - 1].length"
                                                :class="playTour == i ? 'icon-pause' : 'icon-play'"
                                                :title="playTour == i ? $t('pause'): $t('play')"
                                                @click="send_tour(i)"></el-button>
											<el-button v-show="isSupportPtzSettings"
												class="icon-setting1"
                                                :title="$t('set')"
												@click="set_tour(i)"
											></el-button>
                                            <el-button v-show="isSupportPtzSettings && tourArr[i - 1].length"
                                                class="icon-tour_del"
                                                :title="$t('delete')"
                                                @click="del_tour(i)"></el-button>
										</span>
									</li>
								</ul>
							</el-tab-pane>
							<el-tab-pane name="ptzPattern" v-if="!isFisheye">
								<span slot="label" class="icon-pattern" :title="$t('pattern')"></span>
								<ul
									class="infinite-list"
									style="overflow:auto"
								>
									<li
										v-for="i in PATTERN_NUM"
										class="infinite-list-item"
										:key="i"
									>
										<label>{{ format_preset_id(i, 3) }}</label>
										<label class="pattern-name">{{ `${$t('pattern')} ${i}` }}</label>
										<span>
                                            <el-button v-show="isSupportPtzCtrl && patternArr[i - 1].exist"
                                                :class="playPattern == i ? 'icon-pause' : 'icon-play'"
                                                :title="playPattern == i ? $t('pause'): $t('play')"
                                                @click="send_pattern(i)"></el-button>
											<el-button v-show="isSupportPtzSettings"
												class="icon-tabs-record"
                                                :style="{color: patternArr[i - 1].record ? '#f00' : '#606266'}"
                                                :title="patternArr[i - 1].record ? $t('stopRecord'): $t('startRecord')"
												@click="record_pattern(i)"
											></el-button>
                                            <el-button v-show="isSupportPtzSettings && patternArr[i - 1].exist"
                                                class="icon-tour_del"
                                                :title="$t('delete')"
                                                @click="del_pattern(i)"></el-button>
										</span>
									</li>
								</ul>
							</el-tab-pane>
						</el-tabs>
						<div class="ptz-list" v-if="setTour != -1">
							<div class="tour-header">
								<label>{{`${$t('path')} ${setTour + 1}`}}</label>
								<span>
									<el-button
										class="icon-tour-add" @click="add_tour_content"
                                        :title="$t('add')"
									></el-button>
									<el-button
										class="icon-tour_del" @click="del_tour_content(selectedPid)"
                                        :title="$t('delete')"
									></el-button>
									<el-button class="icon-tour_up" @click="up_tour_content"
                                        :title="$t('up')"
                                        ></el-button>
									<el-button
										class="icon-tour_down" @click="dw_tour_content"
                                        :title="$t('down')"
									></el-button>
								</span>
							</div>
							<ul class="tour-set">
								<li>
									<label class="list-index"></label>
									<label class="preset">{{$t('preset')}}</label>
									<label class="speed">{{$t('speed')}}</label>
									<label class="time">{{$t('time')}}</label>
								</li>
								<li
									v-for="(item, i) in editTour"
									class="infinite-list-item is-error"
									:key="i"
                                    @click="selectedPid = i"
                                    :class="{ 'selected-li': selectedPid == i , 'is-first': error.id == i}"
								>
									<label class="list-index">{{ format_preset_id(i + 1, 2) }}</label>
									<el-select
										class="preset"
										v-model="item.pid"
                                        @click.native="if(selectedPid != i)selectedPid = i"
									>
                                        <el-option :value="item.pid" :label="item.pid" v-if="selectedPid != i"></el-option>
                                        <div v-if="selectedPid == i">
                                            <el-option
                                            v-for="preset in presetOptions"
                                            :key="preset"
                                            :value="preset"
                                            :label="preset"
                                            ></el-option>
                                        </div>
                                    </el-select>
									<el-select
										class="speed"
										v-model="item.speed"
                                        @click.native="if(selectedPid != i)selectedPid = i"
									>   
                                        <el-option :value="item.pid" :label="item.pid" v-if="selectedPid != i"></el-option>
                                        <div v-if="selectedPid == i">
                                            <el-option
                                                v-for="speed in TOUR_MAX_SPEED"
                                                :key="speed"
                                                :value="speed"
                                                :label="speed"
                                            ></el-option>
                                        </div>
                                    </el-select>
									<el-input
										class="time"
										v-model="item.time"
                                        maxlength="3"
                                        @keyup.native="tourTimeLimit(i)"
                                        @blur="tourTimeBlur(i)"
									></el-input>
                                    <div class="el-form-item__error" v-show="error.show">{{error.msg}} </div>
								</li>
							</ul>
							<div class="btn-bar">
								<el-button type="primary" @click="save_tour">{{
									$t("save")
								}}</el-button>								
								<el-button type="primary" @click="cancel_save_tour">{{
									$t("cancel")
								}}</el-button>
							</div>
						</div>
					</div>
				</el-tab-pane>
				<el-tab-pane name="imageTab" v-if="showImagePanel">
					<span slot="label" class="icon-img"></span>
					<div class="imgPanel">
                        <ul class="imgInfo" v-if="showZoomPos">
                            <li class="icon-magnifier" :title="$t('zoom')">
                                <el-slider v-model="zoomPos" :min="0" @change="change_zoom_pos"  show-tooltip="true"></el-slider>
                            </li>
                        </ul>
                        <ul class="imgInfo" v-if="isAutoFNew&&!showPtzPanel">
                            <li :class="{'icon-ptzSpeed btn-border': true, 'disableClick': !isSupportPtzCtrl}"
                                :title="$t('focusSpeed')">
                                <el-slider :min="1" :max="10" v-model="ptzSpeed" @change="change_ptz_speed"  show-tooltip="true"></el-slider>
                            </li>
                        </ul>
                        <ul class="ptz-ctrl focus-ctrl" v-if="!showPtzPanel">
							<li :class="{'icon-zoom1 btn-border': true, 'disableClick': !isSupportPtzCtrl}"
                                v-if="isAutoFNew"
                                :title="$t('zoomOut')"
                                @mousedown="send_ptz_cmd(4)" @mouseup="send_ptz_stop_cmd" @mouseout="ptz_mouse_out"></li>
							<li :class="{'icon-zoom2 btn-border': true, 'disableClick': !isSupportPtzCtrl}"
                                v-if="isAutoFNew"
                                :title="$t('zoomIn')"
                                @mousedown="send_ptz_cmd(3)" @mouseup="send_ptz_stop_cmd" @mouseout="ptz_mouse_out"></li>
							<li :class="{'icon-focusminusb btn-border': true, 
                                'disableClick': (!isSupportPtzCtrl || disableFocus)}"
                                v-if="isAutoF"
                                :title="$t('focusOut')"
                                @mousedown="send_ptz_cmd(2)" @mouseup="send_ptz_stop_cmd" @mouseout="ptz_mouse_out"></li>
							<li :class="{'icon-focusplusb btn-border': true, 
                                'disableClick': (!isSupportPtzCtrl || disableFocus)}"
                                v-if="isAutoF"
                                :title="$t('focusIn')"
                                @mousedown="send_ptz_cmd(1)" @mouseup="send_ptz_stop_cmd" @mouseout="ptz_mouse_out"></li>
                            <li class="icon-irisminusb btn-border" v-if="!noManualIris"
                                :title="$t('irisOut')"
                                @mousedown="send_iris(0)" @mouseup="send_ptz_stop_cmd" @mouseout="ptz_mouse_out"></li>
							<li class="icon-irisplusb btn-border" v-if="!noManualIris"
                                :title="$t('irisIn')"
                                @mousedown="send_iris(1)" @mouseup="send_ptz_stop_cmd" @mouseout="ptz_mouse_out"></li>
							<li class="icon-default" v-if="showLensDiv" @mousedown="reset_focus"
                                :title="$t('lensInit')"></li>
							<li class="icon-deffocb" v-if="showLensDiv" @mousedown="send_ptz_cmd(16)"
                                :title="$t('auxilyFocus')"></li>
							<li class="icon-autoiris"
                                v-if="showLensDiv"
                                :class="{'in-effect': irisMode,
                                    'locked-active': irisMode,
                                    'locked-disabled': !irisMode}"
                                :title="$t('autoIris')"
                                @click="change_iris_mode"></li>
						</ul>                       
						<ul class="imgInfo" v-if="liveVideoPri">
							<li class="icon-bright" :title="$t('brightness')">
								<el-slider v-model="bright" @change="change_bright" show-tooltip="true"></el-slider>
							</li>
							<li class="icon-contrast" :title="$t('contrast')">
								<el-slider v-model="contrast" @change="change_contrast" show-tooltip="true"></el-slider>
							</li>
							<li class="icon-saturation" :title="$t('saturation')">
								<el-slider v-model="saturation" @change="change_saturation" show-tooltip="true"></el-slider>
							</li>
							<li class="icon-sharp" :title="$t('sharp')">
								<el-slider v-model="sharp" @change="change_sharp" show-tooltip="true"></el-slider>
							</li>
							<li class="icon-dnr-2d" :title="$t('dnr2d')">
								<el-slider v-model="dnr2d" @change="change_dnr2d" show-tooltip="true"></el-slider>
							</li>
							<li class="icon-nflevel" :title="$t('dnr3d')">
								<el-slider v-model="nflevel" @change="change_nflevel" show-tooltip="true"></el-slider>
							</li>
							<li class="btn-box">
								<el-button
									type="primary"
									class="msButton"
									size="mini"
                                    @click="click_default"
									>{{ $t("default") }}</el-button
								>
							</li>
						</ul>
					</div>
				</el-tab-pane>
			</el-tabs>
		</div>
        <el-dialog
			:title="showDialogForm ? $t('add') : $t('lpr')"
			:visible.sync="showDialog"
			:close-on-click-modal="false"
            center="true"
            :width="showDialogForm ? '30%' : '50%'"
		>
            <el-form :model="diaInfo" label-width="auto" size="mini" v-if="showDialogForm">
                <el-form-item :label="$t('licensePlate')">
                    <el-input v-model="diaInfo.plate" :style="{direction: isIran ? 'rtl' : 'ltr'}" :disabled="true"
                    ></el-input>
                </el-form-item>
                <el-form-item :label="$t('type')">
                    <el-select v-model="diaInfo.type">
                        <el-option
                            v-for="item in plateTypes"
                            :key="item.value"
                            :value="item.value"
                            :label="item.name"
                        ></el-option>
                    </el-select>
                </el-form-item>
			</el-form>
            <div slot="footer" v-if="showDialogForm">
                <el-button type="primary" @click="add_lpr_list"
                    >{{ $t("save") }}</el-button>
                <el-button type="primary" plain @click="showDialog = false">{{
                    $t("cancel")
                }}</el-button>
            </div>
            <img :src="diaInfo.imgSrc" v-if="!showDialogForm" class="image-container"/>
        </el-dialog>
        <el-dialog
			:title="$t('faceDetection')"
			:visible.sync="showFaceDetail"
			:close-on-click-modal="false"
            :width="'700px'"
		>
            <div >
                <div class="face-detail-time">
                    <label>{{`${$t('capturTime')}`}}</label>:
                    <span v-text="curCaptureTime"></span>
                </div>
                <div class="attribute" style='margin-left:40px;'>
                    <div class="facePic" :style="{backgroundImage: facePicUrl}"
                        :class="{'face-bottom': faceBkgUrl.length == 0}"
                    >
                    </div>
                    
                    <div>
                        <ul>
                            <li>
                                <label>{{`${$t('gender')}`}}</label>:
                                <span v-text="curGender"></span>
                            </li>
                                
                            <li>
                                <label>{{`${$t('age')}`}}</label>:
                                <span v-text="curAge"></span>
                            </li>
                            <li>
                                <label>{{`${$t('glasses')}`}}</label>:
                                <span v-text="curGlasses"></span>
                            </li>
                            
                            <li>
                                <label>{{`${$t('mask')}`}}</label>:
                                <span v-text="curMask"></span>
                            </li>
                            
                            <li>
                                <label>{{`${$t('cap')}`}}</label>:
                                <span v-text="curCap"></span>
                            </li>
                        </ul>
                    </div>
                </div>
                <div align="center" class="backgroundPic" 
                :style="{backgroundImage: faceBkgUrl}"
                :class="{'face-bottom': faceBkgUrl.length > 0}"
                v-show="faceBkgUrl.length > 0"
                >
                </div>
            </div>
        </el-dialog>
	</div>
</template>
<script>
import LiveSelect from "@/components/live-select"
import { get_config_data, send_vb_htm, set_page_data, get_reload_flag } from '@/api/page';
import { var_start, loadOBJ, PlayActiveXM, is_anony, Judge_Browser_Ver, xPlayer, IsNull, browser_IE, init_player_paint,
    CheckBrowserType, check_over_max_conn, is_fisheye_software_correction, GetIpAddr, GetCamPlayerUrl, hide_right_menu,
    unInitPlayer, IsForbidLiveViewFeature
} from '@/utils/var'
import playerApi from '@/utils/player-api'
import { get_sys_info } from '@/utils/sys-info'
import { fishDisplayMode, fishInstallMode, fishCorrectMode, send_ptz_cmd_to_device, get_length, sub_string_x,
    show_save_successful, show_loading, alarmEvent, start_online_heartbeat, send_ptz_cmd_to_device_str, vcaEnable,
    is_support_audio_settings, is_support_rtsp_access, is_support_live_view, show_warning_msg, is_viewer,
    is_support_lpr_permission, is_support_event_permission, is_support_ptz_control, is_support_ptz_settings,
    is_support_fisheye_settings, liveview_enable_play_audio  } from '@/utils/common'
import { ASTGUI } from '@/utils/astman';
import { tipsBox, confirmBox, isNull, msgBox } from '@/utils/utils'
import { ptzQuick, set_cur_stream, set_manual_track, fishStreamType, Get3dPosButtonUp, GetZoomCmd } from '@/utils/ptz-common'
import { Message } from 'element-ui'
import {pwmType} from "../../utils/common";
export default {
	data() {
        let faceTableArray = [];
        for (let i = 0; i < 20; i++) {
            let item = {};
            item.src = '';
            item.genderClass = 'icon-female';
            item.genderTitle = this.$t('female');
            item.ageClass = 'icon-adult';
            item.ageTitle = this.$t('adult');
            item.glassClass = 'icon-glasses';
            item.glassTitle = this.$t('yes');
            item.maskClass = 'icon-mask';
            item.maskTitle = this.$t('yes');
            item.time = '';
            item.show = false;
            faceTableArray.push(item);
        }
		return {
            LPR_ROI_NUM: 4,
            LPR_PRESET_NUM: 4,
            PRESET_NUM: 300,
            PATROL_NUM: 8,
            PATTERN_NUM: 4,
            TOUR_MAX_PRESET_NUM: 48,
            TOUR_MAX_SPEED: 40,
			streamOptions: [
				{ value: 0, label: this.$t("primaryStream") }
			],
            protocolOptions: [
				{ value: 0, label: "UDP" },
                { value: 1, label: "TCP" },
                { value: 2, label: "HTTP" }
			],
            smoothOptions: [
				{ value: 0, label: this.$t('leastDelay') },
                { value: 1, label: this.$t('balance') },
                { value: 2, label: this.$t('bestFluency') }
			],
            overlayOptions: [
                { value: 0, label: this.$t('hideRegion') },
                { value: 1, label: this.$t('regionEntrance') },
                { value: 2, label: this.$t('regionExiting') },
                { value: 3, label: this.$t('advancMotion') },
                { value: 6, label: this.$t('lineCross') },
                { value: 4, label: this.$t('loitering') },
                { value: 8, label: this.$t('peopleCount') },
                { value: 7, label: this.$t('objectLeftRemove') },
                { value: 9, label: this.$t('viewingArea') },
                { value: 10, label: this.$t('regionalCounter') }
            ],
            lprOptions: [
                { value: 0, label: this.$t('hideRegion') },
                { value: 1, label: this.$t('lpr') }
            ],
            faceOptions: [
                { value: 0, label: this.$t('hideRegion') },
                { value: 1, label: this.$t('faceDetect') }
            ],
            isIE: browser_IE,
            showRight: true,
            showStream: true,
            showOverlay: true,
            overlaySel: 0,
            lprSel: 0,
            faceSel: 0,
			streamSel: 0,
			protocolSel: 0,
            smoothSel: 0,
			scaleMode: "icon-autosize",
			scaleSel: "AUTO",
			activeTab: "imageTab",
            zoomPos: 0,
            showZoomPos: true,
            isAutoFNew: false,
            isAutoF: true,
            irisTimer: null,
            bright: 0,
            contrast: 0,
            saturation: 0,
            sharp: 0,
            dnr2d: 0,
            nflevel: 0,
            showCanvas: true,
            show_max_conn_tip: false,
            isPlaying: 0,
            playAudio: 0,
            playTalk: 0,
            showMute: false,
            showSpeaker: false,
            isRecording: 0,
            zoomEnable: 0,
            sysInfo: {},
            recordTime: 0,
            pos_3d_enable: 0,
            mainSmartStream: 0,
            subSmartStream: 0,
            thirdSmartStream: 0,
            fishDisplay: 0,
            fishInstall: 0,
            fishCorrect: 0,
            curFishInstall: 0,
            isRecordingFish: [],
            fishCurWnd: 0,
            trackEnable: 0,
            showTrack: 0,
            autoTrackTimer: 0,
            autoTrackTimerFlag: false,
            drawTimer: 0,
            drawTimerFlag: false,
            vcaRect: '',
            trackRect: '',
            intrusionEnterPlgX: [],
            intrusionEnterPlgY: [],
            inrtusionEnterPresetX: [[],[],[],[]],
            inrtusionEnterPresetY: [[],[],[],[]],
            intrusionExitPresetX: [[],[],[],[]],
            intrusionExitPresetY: [[],[],[],[]],
            adMotionPresetX: [[],[],[],[]],
            adMotionPresetY: [[],[],[],[]],
            loiteringPresetX: [[],[],[],[]],
            loiteringPresetY: [[],[],[],[]],
            objectPresetX: [[],[],[],[]],
            objectPresetY: [[],[],[],[]],
            intrusionExitPlgX: [],
            intrusionExitPlgY: [],
            adMotionPlgX: [],
            adMotionPlgY: [],
            loiteringPlgX: [],
            loiteringPlgY: [],
            objectPlgX: [],
            objectPlgY: [],
            intrusionEnterEnable: [],
            intrusionExitEnable: [],
            adMotionEnable: [],
            loiteringEnable: [],
            objectEnable: [],
            lineEnable: [],
            crossLine: [],
            crossLineDir: [],
            crossLinePreset: [],
            crossLineDirPreset: [],
            countEnable: [],
            countLine: [],
            countLineDir: [],
            countLinePreset: [],
            countLineDirPreset: [],
            regionalPeopleEnable: [],
            regionalPeoplePlgX: '',
            regionalPeoplePlgY: '',
            regionalPeoplePlgXPreset: [],
            regionalPeoplePlgYPreset: [],
            mainResW: 0,
            mainResH: 0,
            subResW: 0,
            subResH: 0,
            triResW: 0,
            triResH: 0,
            subStreamEnable: 0,
            triStreamEnable: 0,
            fourStreamEnable: 0,
            fifStreamEnable: 0,
            activePTZ: 'ptzPreset',
            manualTrack: 0,
            ptzSpeed: 5,
            irisMode: 0,
            irisType: 1,
            noManualIris: false,
            noAutoIris: false,
            lightMode: 0,
            lightTimer: 0,
            posEnable: 0,
            patrolMode: 0,
            patrolTimer: 0,
            watchEnable: 0,
            watchTime: 0,
            watchMode: 0,
            watchId: 0,
            showDefog: true,
            manualDefog: 0,
            defogTimer: null,
			setTour: -1,
            playTour: 0,
            fishPlayTour: [0, 0, 0, 0, 0],
			presetTable: [{ name: 'Preset1' }],
            tourArr: [[], [], [], [], [], [], [], []],
            tourList: [[], [], [], [], [], [], [], []],
            fishTourList: [],
			faceTable: faceTableArray,
			presetArr: [],
            presetName: [],
            editPreset: -1,
            selectedPid: -1,
            editTour: [],
            tourTimer: null,
            tourTimerFlag: false,
            patternArr: [{}, {}, {}, {}],
            playPattern: 0,
            patternTimer: null,
            autoScanStatus: false,
            autoScanTimer: null,
            isLpr: false,
            isLprMode: false,
            LPR_MAX_ROW: 100,
            lprData: [],
            lprType: ['-', this.$t('blackList'), this.$t('whiteList'), this.$t('visitor')],
            lprDirec: ['-', this.$t('approach'), this.$t('away')],
            lprColor: ['-', this.$t('black'), this.$t('blue'), this.$t('cyan'),
                this.$t('gray'), this.$t('green'), this.$t('red'), this.$t('white'),
                this.$t('yellow'), this.$t('purple'), this.$t('orange')],
            vehicleType: ['-', this.$t('car'), this.$t('motorcycle'), this.$t('bus'),
                this.$t('truck'), this.$t('minibus')],
            lprInfo: {},
            hasLprPermission: true,
            plateImgSrc: '',
            plateLoadEvent: null,
            lprScene: 0,
            lprRoiRegion: [],
            lprRoiRegionPreset: [],
            curPresetPos: 1,
            vcaCurPreset: -1,
            lprWidth: 0,
            lprHeight: 0,
            lprStatus: 0,
            lprSeparator: '',
            lprSnapSort: '',
            lprSnapPos: '',
            lprSnapId: '',
            deviceName: '',
            isIran: false,
            lprSocket: null,
            webSocketObject: null,
            lprLogsCnt: 0,
            lprFilePrefix: '',
            lastLprData: {},
            showDialog: false,
            showDialogForm: false,
            visible: false,
            isStart: false,
            isStart1: false,
            showExt: false,
            hasExtout: true,
            alarmType: -2,
            alarmTypeEx: -2,
            alarmDuration: 1,
            alarmDurationEx: 1,
            patrolRecover: false,
            plateTypes: [
				{ value: 1, name: this.$t("blackList") },
				{ value: 2, name: this.$t("whiteList") }
			],
            diaInfo: {
                plate: '',
                type: 0,
                imgSrc: ''
            },
            isLoaded: false,
            getFishTime: 0,
            getPresetTime: 0,
            vcaenable: 0,
            showAlarm: false,
            showRecordAlarm: false,
            showScheAlarm: false,
            showMotion: false,
            showNewAlarm: false,
            showRadarAlarm: false,
            showIotAlarm: false,
            showCountAlarm: false,
            showLprAlarm: 0,
            lprLastestType: 0,
            alarmTimer: null,
            alarmTimerFlag: true,

            FLG_UI_TASK: 1 << 10,
            FLG_UI_VCA_INTRUSION_ENTER: 1 << 11,
            FLG_UI_RADAR: 1 << 12,
            FLG_UI_VCA_LOITERING :1 << 13,
            FLG_UI_VCA_ADVANCED_MOTION :1 << 14,
            FLG_UI_VCA_LINECROSSING1 :1 << 15,
            FLG_UI_VCA_CTD  :1 << 16,
            FLG_UI_VCA_INTRUSION_EXIT :1 << 17,
            FLG_UI_VCA_LINECROSSING2 :1 << 18,
            FLG_UI_VCA_LINECROSSING3 :1 << 19,
            FLG_UI_VCA_LINECROSSING4 :1 << 20,
            FLG_UI_VCA_HUMAN :1 << 21,
            FLG_UI_VCA_COUNTING :1 << 22,
            FLG_UI_VCA_OBJECT_LEFT : 1 << 25,
            FLG_UI_VCA_OBJECT_REMOVE : 1 << 26,
            FLG_UI_IOT :1 << 23,
            FLG_UI_LPR_BLACK :1 << 27,
            FLG_UI_LPR_WHITE :1 << 28,
            FLG_UI_LPR_VISITOR :1 << 29,
            FLG_UI_IP_CONFLICT : 1 << 8,
            FLG_UI_RECORD_FAIL : 1 << 32,
            FLG_UI_SD_FULL 	   : 1 << 1,
            FLG_UI_SD_UNINIT   : 1 << 2,
            FLG_UI_SD_ERROR    : 1 << 3,
            FLG_UI_NO_SD 	   : 1 << 4,

            showFishPanel: false,
            showPtzPanel: false,
            showImagePanel: true,
            isFaceMode: false,
            facePointTimer: null,
            facePicTimer: null,
            faceAttibuteData: [],
            showFaceDetail: false,
            faceBkgUrl: '',
            drawDataTimer: 0,
            drawDataTimerFlag: false,
            isFisheye: false,
            curWndPlay: [],
            playUrlIndex: [-1, -1, -1, -1, -1],
            showDewarpBox: true,
            curDisplay: 0,
            lastSwitchTime: 0,
            switchInterval: 5,
            loading: false,
            isLoadingView: 0,
            channelPlay: [1],
            curConn: 10,
            recordStartFish: [0, 0, 0, 0, 0, 0, 0, 0, 0],
            curWndZoom: [0, 0, 0, 0, 0, 0, 0, 0, 0],
            stopAllFishVideo: 0,
            switchFlag: 0,
            fishSubEnable: [],
            nowStream: 0,
            showFishDisplaySel: false,
            windowsModel: 0,
            lastWindowsModel: 0,
            updatePreset: 0,
            updateFishView: 0,
            lastStream: -2,
            curWndChannel: 0,
            selectedChannel: 0,
            fishPresetArr: [[], [], [], [], []],
            fishPresetName: [[], [], [], [], []],
            FISH_MAX_VIEW_NUM: 5,
            ptzTimer: null,
            bPtzTimer: 0,
            fishPtzRegion: '',
            fishAutoScan: [0, 0, 0, 0, 0],
            fishTourTimer: null,
            fishTourTimerFlag: false,
            fishView: [],
            displayDisablePtz: false,
            disableSubChannel: false,
            curUrlListIndex: -1,
            enableAudio: false,
            getStateTimer: null,
            getStateFlag: false,
            maxConn: 10,
            limitPlayFailed: 0,
            setLoadingTimes: 0,
            channelShowSub: false,
            smartStreamArr: [0, 0, 0, 0, 0],
            osdZmTime: 0,
            osdMdTime: 0,
            osdPatrolTime: 0,
            osdScanTime: 0,
            disableTrack: false,
            smartEventEnable: 0,
            lprEnable: 0,
            dewarpType: 0,
            displayArr: [],
            displayTitleArr: ['1O', '1P', '2P', '2R', '4R', '1O3R', '1P3R', '1O1P3R', '1P1R', '1P4R', '1P6R', '1O8R'],
            showPlus: true,
            showDisplay1O3R: true,
            G_SOFT_DEWARP: 1,
            lastFishMove: 0,
            ptzIsUsing: 0,
            patrolRecoveryEnable: 0,
            windowIcon: 'icon-play2',
            showFishWindow: true,
            showVcaInstall: false,
            vcaInstall: 0,
            faceMSCEnable: false,
            playActiveTimer: null,
            fishScaleTimer: null,
            overLayTimer: null,
            switchLoadingTimer: null,
            subSmartTimer: null,
            lprLayTimer: null,
            faceLayTimer: null,
            showLensDiv: false,
            isAnony: false,
            firstRadarTips: true,
            curcorridormode: 0,
            curimagerotation: 0,
            disablePTZFeature: false,
			isShowLight: false,
            liveVideoPri: true, // 是否视频查看权限
            showTriggleRightBtn: true,
            showFaceBtn: true,
            isSupportPtzSettings: true,
            isSupportPtzCtrl: true,
            isForbidPtzCtrlAndSettings: false,
            countLineAlarm: [0, 0, 0, 0],
            entrancescene: 0,
            exitscene: 0,
            motionscene: 0,
            loiteringscene: 0,
            leftremovescene: 0,
            linecrossingscene: 0,
            regionalpeoplescene: 0,
            countlinescene: 0,
            iotSupport: 0,
            disableZoom: false,
            disableFocus: false,
            disableLens: false,
            disableAutoFocus: false,
            disableAutoIris: false,
            curCap: '',
            manualoutputSupport: true,
            hasDestroy: false,
            disableFocusTimer: null,
            error: {
                show: false,
                msg: ''
            }
		}
	},
	components: {
		LiveSelect
	},
    computed: {
        selProtocolName () {
            let name = ''
                for (const item of this.protocolOptions) {
                if (item.value == this.protocolSel) {
                    name = item.label
                    break
                }
            }
            return name
        },
        selStreamName () {
            let name = ''
            for (const item of this.streamOptions) {
                if (item.value == this.streamSel) {
                    name = item.label
                    break
                }
            }
            return name
        },
        presetOptions () {
            let a = [];
            for (let i=1; i<=this.PRESET_NUM; i++) {
                if (i == ptzQuick.PRESET_START) {
                    i = ptzQuick.PRESET_END;
                    continue;
                }
                a.push(i);
            }
            return a;
        },
        overlayOptionsLable() {
            let i = 0;
            for (i = 0; i < this.overlayOptions.length; i++) {
                if (this.overlayOptions[i].value == this.overlaySel) {
                    break;
                }
            }
            if (i >= 0 && i < this.overlayOptions.length) {
                return this.overlayOptions[i].label;
            } else {
                return this.overlayOptions[0].label;
            }
        }
    },
	methods: {
        reset_focus () {
            this.send_ptz_cmd(17)
            this.zoomPos = 100 // 初始化后，镜头放大到最大
        },
        is_support_protocol (val) {
            for (const item of this.protocolOptions) {
                if (item.value == val) {
                    return true
                }
            }
            return false
        },
        is_forbid_ptz_fisheye_pri() {
            return !is_support_ptz_control() && !is_support_ptz_settings() && !is_support_fisheye_settings()
        },
        click_output_stus (dOutIndex) {
            let openStus = 0;
            let url = '';
            if (dOutIndex == 0) {
                if (this.isStart == true) {
                    openStus = 1;
                }
                url = `&outputstatus=${openStus}&outputactiontime=${this.alarmType}`;
                if (this.alarmType != -2) {
                    this.isStart = false;
                }
                if (this.alarmType == -1) {
                    url += `&outputduration=${this.alarmDuration}`;
                }
            } else {
                if (this.isStart1 == true) {
                    openStus = 1;
                }
                url = `&outputstatusex=${openStus}&outputactiontimeex=${this.alarmTypeEx}`;
                if (this.alarmTypeEx != -2) {
                    this.isStart1 = false;
                }
                if (this.alarmTypeEx == -1) {
                    url += `&outputdurationex=${this.alarmDurationEx}`;
                }
            }
            set_page_data(`alarm.${alarmEvent.ALARM_EVENT_DINPUT}`, url);
        },
		change_stream (val) {
            if (val == this.nowStream)
                return ;
			this.nowStream = this.streamSel = val;
            this.showCanvas = true;
            this.show_max_conn_tip = false;
            this.isPlaying = 1;
            // document.getElementsByClassName('selCanvas')[0].style.display = '';
            if (this.isRecording) {
                this.click_record();
            }
            playerApi.MsDisConnect();
            ASTGUI.cookies.setCookie('msStreamIndex', this.streamSel + '');
            this.change_scale(2);
            playerApi.MsSetParam("hasclick", "0");	//for winplugin
            if (this.scaleSel == '100%')
                this.change_scale(1);
            else
                this.change_scale(0);
            if (this.zoomEnable) {
                this.zoomEnable = 0;
                playerApi.MsSetParam('zoomenable', '0');
            }
            if (this.playAudio)
                this.click_audio();
            this.load_overlays();
            set_cur_stream(val)
		},
		change_scale (val) {
			let obj = playerApi.MsGetPlayer();
            if (Judge_Browser_Ver() === 0) {
                obj = document.getElementById('PlayerActiveX');
            }
            // let width = parseInt(this.$refs.video.offsetWidth), height = parseInt(this.$refs.video.offsetHeight);
            let width = parseInt(this.$refs.canvas.offsetWidth), height = parseInt(this.$refs.canvas.offsetHeight);
            if (this.isLprMode) {
                width = parseInt(this.$refs.video.offsetWidth);
                height = parseInt(this.$refs.video.offsetHeight);
            }
            if (val == 0) {
                this.scaleMode = 'icon-autosize';
                this.scaleSel = 'AUTO';
                this.$refs.scale.labelText = 'AUTO';
            } else if (val == 1) {
                this.scaleMode = 'icon-a-100';
                this.scaleSel = '100%';
                this.$refs.scale.labelText = '100%';
                switch (this.streamSel) {
                    case 0:
                        width = this.mainResW;
                        height = this.mainResH;
                        break;
                    case 1:
                        width = this.subResW;
                        height = this.subResH;
                        break;
                    case 2:
                        width = this.triResW;
                        height = this.triResH;
                        break;
                }
                if (this.sysInfo.isfisheye == '1') {
                    let url;
                    if (this.fishCorrect == fishCorrectMode.MUL_STREAM)
                        url = `&getfishResolution=${this.get_channel_index()}`;
                    else
                        url = `&getfishResolution=${this.streamSel}`;
                    set_page_data('fishresolution', url).then( res => {
                        let ix = res.indexOf('=');
                        result = ix >= 0 ? res.substr(ix+1, res.lastIndexOf(';')) : '';
                        let streamParam = res.split(';');
                        if (streamParam[0] > 0) {
                            switch (this.windowsModel) {
                                case 0:
                                    width = streamParam[0];
                                    height = streamParam[1];
                                    break;
                                case 1:
                                    width = 2 * streamParam[0];
                                    height = 2 * streamParam[1];
                                    break;
                                case 2:
                                    if (this.fishCurWnd > 0)
                                        width = 2 * streamParam[0];
                                    else
                                        width = streamParam[0];
                                    height = 3 * streamParam[1];
                                    break;
                            }
                            obj.width = width;
                            obj.height = height;
                            this.$refs.video.style.width = `${width}px`;
                            this.$refs.video.style.height = `${height}px`;
                        }
                    })
                }
            }
            console.log(this.scaleSel)
            if (this.scaleSel == 'AUTO') {
                this.$refs.video.style.width = `100%`;
                this.$refs.video.style.height = `100%`;
                obj.style.width = `100%`;
                obj.style.height = `100%`;
            } else {
                this.$refs.video.style.width = `${width}px`;
                this.$refs.video.style.height = `${height}px`;
                obj.style.width = `${width}px`;
                obj.style.height = `${height}px`;
            }
            let canvasH = this.$refs.canvas.offsetHeight;
            if (canvasH > height)
                this.$refs.video.style.marginTop = canvasH / 2 - height / 2;
            else
                this.$refs.video.style.marginTop = 0;
            if (val == 2) {
                this.playActiveTimer = setTimeout( () => {
                    PlayActiveXM(this.streamSel, this.get_smart_stream(this.streamSel),
                        this.update_select_wnd_status, this.get_fish_mouse_move, this.set_correct_model, this);
                }, 50);
                if (this.sysInfo.isfisheye == '1' && this.fishCorrect == fishCorrectMode.JOINT_VIDEO) {
                    this.fishScaleTimer = setTimeout(() => {
                        let talk = this.playTalk;
                        playerApi.MsSetParam('fisheye', '1');
                        if (this.sysInfo.iscv2xplatform == '1')
                            playerApi.MsSetParam("fishcorrectplatform", "1");
                        this.set_correct_model(this.fishDisplay);
                        playerApi.MsSetParam('fishinstallmodel', this.fishInstall);
                        playerApi.MsSetParam('fishdisplayplaymodel', this.fishDisplay);
                        this.update_select_wnd_status(0);
                        this.set_channel_play_state(this.nowStream);
                        if (!this.bPtzTimer)
                            this.get_fish_ptz_region();
                        playerApi.MsSetParam('livevideoui', '1');
                        if (talk == 1 && this.playTalk == 0)
                            this.click_talk();
                    }, 100);
                }
            }
		},
        do_full_screen () {
            if (this.isPlaying)
                playerApi.MsSetParam('fullscreen', '1');
        },
        get_smart_stream (stream) {
            switch (stream) {
                case 0:
                    return this.mainSmartStream;
                case 1:
                    return this.subSmartStream;
                case 2:
                    return this.thirdSmartStream;
                default:
                    return this.mainSmartStream;
            }
        },
        changeProtocol (val) {
            this.protocolSel = val;
            playerApi.MsSetParam('protocol', val);
            location.reload();
        },
        changeSmooth (val) {
            this.smoothSel = val;
            playerApi.MsSetParam('smoothstream', val);
            location.reload();
        },
        load_overlays () {
            if (this.isusingcropvideo) {
                if (this.streamSel != 0) {
                    this.showOverlay = false;
                    playerApi.MsSetMotionMap(1,'0');
                    this.hide_all_lines();
                    this.clear_auto_track();
                }
            }
            if (this.sysInfo.supportvca != '1' || this.isFaceMode)
                this.showOverlay = false;
            else {
                if ((this.sysInfo.isfisheye == '1' && this.fishDisplay == fishDisplayMode.DISPLAY_1O) || this.sysInfo.isfisheye == '0')
                    this.showOverlay = true;
                let vcaType = ASTGUI.cookies.getCookie("vcaType");
                this.overlaySel = IsNull(vcaType) ? 0 : parseInt(vcaType);
                if (this.overLayTimer != null) {
                    clearTimeout(this.overLayTimer);
                }
                if (this.sysInfo.isusingcropvideo == '1' && this.overlaySel == 9)
                    this.overLayTimer = setTimeout(() => {
                        this.change_overlay(this.overlaySel);
                    }, 500);
                else
                    this.overLayTimer = setTimeout(() => {
                        this.change_overlay(this.overlaySel);
                    }, 4000);
            }
        },
        change_overlay (val) {
            if (!this.showOverlay)
                return ;
            this.overlaySel = val;
            ASTGUI.cookies.setCookie('vcaType', this.overlaySel);
            playerApi.MsSetParam('preview','1');
            playerApi.MsSetPolygonMap(0, 1, '-1:-1:-1:-1:-1:-1:-1:-1:-1:-1', '-1:-1:-1:-1:-1:-1:-1:-1:-1:-1');
            playerApi.MsSetCoutLineData("-1:-1:-1:-1;-1:-1:-1:-1;-1:-1:-1:-1;-1:-1:-1:-1;");
            playerApi.MsSetMotionMap(1,'0');
            playerApi.MsSetRegionAlarm("0:0:-1:-1:-1:-1");
            this.hide_all_lines();
            if(this.sysInfo.isusingcropvideo == '1')
                this.clear_auto_track();
            this.drawTimerFlag = false;
            clearTimeout(this.drawTimer);
            this.drawDataTimerFlag = false;
            clearTimeout(this.drawDataTimer);
            this.draw_liveview_rect(0, '0:0:0:0:0:0:');
            playerApi.MsSetPersonRectRegion(1, "0:0:0:0:0:0");
            playerApi.MsSetPersonRectWithTime(1, '');
            playerApi.MsSetParam('regionindex', -1);
            let plgX,plgY;
            let curPreset = this.vcaCurPreset;
            switch (this.overlaySel) {
                case 1:
                    if (this.sysInfo.isAiPlatform) {
                        playerApi.MsSetParam("regionindex", 4);
                        if (curPreset >= 1 && curPreset <= 4 && this.entrancescene == 1) {
                            curPreset = curPreset - 1;
                            plgX = this.inrtusionEnterPresetX[curPreset][0] + ";" + 
                                this.inrtusionEnterPresetX[curPreset][1] + ";" +
                                this.inrtusionEnterPresetX[curPreset][2] + ";" + 
                                this.inrtusionEnterPresetX[curPreset][3] + ";";
                            plgY = this.inrtusionEnterPresetY[curPreset][0] + ";" + 
                                this.inrtusionEnterPresetY[curPreset][1] + ";" +
                                this.inrtusionEnterPresetY[curPreset][2] + ";" + 
                                this.inrtusionEnterPresetY[curPreset][3] + ";";
                            playerApi.MsSetPolygonMap(1, 4, plgX, plgY);
                        } else {
                            plgX = this.intrusionEnterPlgX[0] + ";" + 
                                this.intrusionEnterPlgX[1] + ";" +
                                this.intrusionEnterPlgX[2] + ";" + 
                                this.intrusionEnterPlgX[3] + ";";
                            plgY = this.intrusionEnterPlgY[0] + ";" + 
                                this.intrusionEnterPlgY[1] + ";" +
                                this.intrusionEnterPlgY[2] + ";" + 
                                this.intrusionEnterPlgY[3] + ";";
                            if (this.entrancescene == 1) {
                                playerApi.MsSetPolygonMap(0, 1, '-1:-1:-1:-1:-1:-1:-1:-1:-1:-1',
                                    '-1:-1:-1:-1:-1:-1:-1:-1:-1:-1');
                            } else {
                                playerApi.MsSetPolygonMap(1, 4, plgX, plgY);
                            }
                        }
                        
                        for (let i=0; i<4; i++) {
                            if (this.intrusionEnterEnable[i]) {
                                this.drawTimerFlag = true;
                                this.drawTimer = setTimeout(() => {
                                    this.update_person(0);
                                }, 100);
                                break;
                            }
                        }
                    } else {
                        playerApi.MsSetPolygonMap(1, 1, this.intrusionEnterPlgX[0], this.intrusionEnterPlgY[0]);
                    }
                    break;
                case 2:
                    if (this.sysInfo.isAiPlatform) {
                        playerApi.MsSetParam("regionindex", 4);
                        if (curPreset >= 1 && curPreset <= 4 && this.exitscene == 1) {
                            curPreset = curPreset - 1;
                            plgX = this.intrusionExitPresetX[curPreset][0] + ";" +
                                this.intrusionExitPresetX[curPreset][1] + ";" +
                                this.intrusionExitPresetX[curPreset][2] + ";" +
                                this.intrusionExitPresetX[curPreset][3] + ";";
                            plgY = this.intrusionExitPresetY[curPreset][0] + ";" +
                                this.intrusionExitPresetY[curPreset][1] + ";" +
                                this.intrusionExitPresetY[curPreset][2] + ";" +
                                this.intrusionExitPresetY[curPreset][3] + ";";
                            playerApi.MsSetPolygonMap(1, 4, plgX, plgY);
                        } else {
                            plgX = this.intrusionExitPlgX[0] + ";" +
                                this.intrusionExitPlgX[1] + ";" +
                                    this.intrusionExitPlgX[2] + ";" +
                                    this.intrusionExitPlgX[3] + ";";
                            plgY = this.intrusionExitPlgY[0] + ";" +
                                this.intrusionExitPlgY[1] + ";" +
                                    this.intrusionExitPlgY[2] + ";" +
                                    this.intrusionExitPlgY[3] + ";";
                            if (this.exitscene == 1) {
                                playerApi.MsSetPolygonMap(0, 1, '-1:-1:-1:-1:-1:-1:-1:-1:-1:-1',
                                    '-1:-1:-1:-1:-1:-1:-1:-1:-1:-1');
                            } else {
                                playerApi.MsSetPolygonMap(1, 4, plgX, plgY);
                            }
                        }
                        for (let i=0; i<4; i++) {
                            if (this.intrusionExitEnable[i]) {
                                this.drawTimerFlag = true;
                                this.drawTimer = setTimeout(() => {
                                    this.update_person(1);
                                }, 100);
                                break;
                            }
                        }
                    } else {
                        playerApi.MsSetPolygonMap(1, 1, this.intrusionExitPlgX[0], this.intrusionExitPlgY[0]);
                    }
                    break;
                case 3:
                    if (this.sysInfo.isAiPlatform) {
                        playerApi.MsSetParam("regionindex", 4);
                        if (curPreset >= 1 && curPreset <= 4 && this.motionscene == 1) {
                            curPreset = curPreset - 1;
                            plgX = this.adMotionPresetX[curPreset][0] + ";" +
                                this.adMotionPresetX[curPreset][1] + ";" +
                                this.adMotionPresetX[curPreset][2] + ";" +
                                this.adMotionPresetX[curPreset][3] + ";";
                            plgY = this.adMotionPresetY[curPreset][0] + ";" +
                                this.adMotionPresetY[curPreset][1] + ";" +
                                this.adMotionPresetY[curPreset][2] + ";" +
                                this.adMotionPresetY[curPreset][3] + ";";
                                playerApi.MsSetPolygonMap(1, 4, plgX, plgY);
                        } else {
                            plgX = this.adMotionPlgX[0] + ";" + this.adMotionPlgX[1] + ";" +
                                this.adMotionPlgX[2] + ";" + this.adMotionPlgX[3] + ";";
                            plgY = this.adMotionPlgY[0] + ";" + this.adMotionPlgY[1] + ";" +
                                this.adMotionPlgY[2] + ";" + this.adMotionPlgY[3] + ";";
                            if (this.motionscene == 1) {
                                playerApi.MsSetPolygonMap(0, 1, '-1:-1:-1:-1:-1:-1:-1:-1:-1:-1',
                                    '-1:-1:-1:-1:-1:-1:-1:-1:-1:-1');
                            } else {
                                playerApi.MsSetPolygonMap(1, 4, plgX, plgY);
                            }
                        }
                        
                        for (let i=0; i<4; i++) {
                           if (this.adMotionEnable[i]) {
                                this.drawTimerFlag = true;
                                this.drawTimer = setTimeout(() => {
                                    this.update_person(2);
                                }, 100);
                                break;
                            }
                        }
                    } else {
                        playerApi.MsSetPolygonMap(1, 1, this.adMotionPlgX[0], this.adMotionPlgY[0]);
                    }
                    break;
                case 4:
                    if (this.sysInfo.isAiPlatform) {
                        playerApi.MsSetParam("regionindex", 4);
                        if (curPreset >= 1 && curPreset <= 4 && this.loiteringscene == 1) {
                            curPreset = curPreset - 1;
                            plgX = this.loiteringPresetX[curPreset][0] + ";" +
                                this.loiteringPresetX[curPreset][1] + ";" +
                                this.loiteringPresetX[curPreset][2] + ";" +
                                this.loiteringPresetX[curPreset][3] + ";";
                            plgY = this.loiteringPresetY[curPreset][0] + ";" +
                                this.loiteringPresetY[curPreset][1] + ";" +
                                this.loiteringPresetY[curPreset][2] + ";" +
                                this.loiteringPresetY[curPreset][3] + ";";
                                playerApi.MsSetPolygonMap(1, 4, plgX, plgY);
                        } else {
                            plgX = this.loiteringPlgX[0] + ";" + this.loiteringPlgX[1] + ";" +
                                this.loiteringPlgX[2] + ";" + this.loiteringPlgX[3] + ";";
                            plgY = this.loiteringPlgY[0] + ";" + this.loiteringPlgY[1] + ";" +
                                this.loiteringPlgY[2] + ";" + this.loiteringPlgY[3] + ";";
                            if (this.loiteringscene == 1) {
                                playerApi.MsSetPolygonMap(0, 1, '-1:-1:-1:-1:-1:-1:-1:-1:-1:-1',
                                    '-1:-1:-1:-1:-1:-1:-1:-1:-1:-1');
                            } else {
                                playerApi.MsSetPolygonMap(1, 4, plgX, plgY);
                            }
                        }
                        for (let i=0; i<4; i++) {
                            if (this.loiteringEnable[i]) {
                                this.drawTimerFlag = true;
                                this.drawTimer = setTimeout(() => {
                                    this.update_person(5);
                                }, 100);
                                break;
                            }
                        }
                    } else {
                        playerApi.MsSetPolygonMap(1, 1, this.loiteringPlgX[0], this.loiteringPlgY[0]);
                    }
                    break;
                case 6:
                    this.show_all_lines();
                    if (this.sysInfo.isAiPlatform) {
                        for (let i=0; i<this.lineEnable.length; i++) {
                            if (this.lineEnable[i] == 1 ) {
                                this.drawTimerFlag = true;
                                this.drawTimer = setTimeout(() => {
                                    this.update_person(4);
                                }, 100);
                                break;
                            }
                        }
                    }
                    break;
                case 7:
                    if (this.sysInfo.isAiPlatform) {
                        playerApi.MsSetParam("regionindex", 4);
                        if (curPreset >= 1 && curPreset <= 4 && this.leftremovescene == 1) {
                            curPreset = curPreset - 1;
                            plgX = this.objectPresetX[curPreset][0] + ";" +
                                this.objectPresetX[curPreset][1] + ";" +
                                this.objectPresetX[curPreset][2] + ";" +
                                this.objectPresetX[curPreset][3] + ";";
                            plgY = this.objectPresetY[curPreset][0] + ";" +
                                this.objectPresetY[curPreset][1] + ";" +
                                this.objectPresetY[curPreset][2] + ";" +
                                this.objectPresetY[curPreset][3] + ";";
                                playerApi.MsSetPolygonMap(1, 4, plgX, plgY);
                        } else {
                            plgX = this.objectPlgX[0] + ";" + this.objectPlgX[1] + ";" +
                                this.objectPlgX[2] + ";" + this.objectPlgX[3] + ";";
                            plgY = this.objectPlgY[0] + ";" + this.objectPlgY[1] + ";" +
                                this.objectPlgY[2] + ";" + this.objectPlgY[3] + ";";
                            if (this.leftremovescene == 1) {
                                playerApi.MsSetPolygonMap(0, 1, '-1:-1:-1:-1:-1:-1:-1:-1:-1:-1',
                                    '-1:-1:-1:-1:-1:-1:-1:-1:-1:-1');
                            } else {
                                playerApi.MsSetPolygonMap(1, 4, plgX, plgY);
                            }
                        }
                        for (let i=0; i<4; i++) {
                            if (this.objectEnable[i]) {
                                this.drawTimerFlag = true;
                                this.drawTimer = setTimeout(() => {
                                    this.update_object_track();
                                }, 100);
                                break;
                            }
                        }
                    } else {
                        playerApi.MsSetPolygonMap(1, 1, this.objectPlgX[0], this.objectPlgY[0]);
                        if (this.objectEnable[0]) {
                            this.drawTimerFlag = true;
                            this.drawTimer = setTimeout(() => {
                                this.update_object_track();
                            }, 100);
                        }
                    }
                    break;
                case 8:
                    this.show_line(5, 0);
                    this.show_line(6, 0);
                    this.show_line(7, 0);
                    this.show_line(8, 0);
                    if (this.countEnable[0] || this.countEnable[1] || this.countEnable[2] || this.countEnable[3]) {
                        this.drawTimerFlag = true;
                        this.drawTimer = setTimeout(() => {
                            this.update_person(7);
                        }, 100);
                        this.drawDataTimerFlag = true;
                        this.drawDataTimer = setTimeout(() => {
                            this.update_count_line_data();
                        }, 100);
                    }
                    break;
                case 9:
                    this.show_auto_track();
                    break;
                case 10:
                    if (this.sysInfo.isAiPlatform) {
                        playerApi.MsSetParam("regionindex", 4);
                        if (curPreset >= 1 && curPreset <= 4 && this.regionalpeoplescene) {
                            curPreset = curPreset - 1;
                            plgX = this.regionalPeoplePlgXPreset[curPreset];
                            plgY = this.regionalPeoplePlgYPreset[curPreset];
                            playerApi.MsSetPolygonMap(1, 4, plgX, plgY);
                        } else {
                            plgX = this.regionalPeoplePlgX;
                            plgY = this.regionalPeoplePlgY;
                            if (this.regionalpeoplescene == 1) {
                                playerApi.MsSetPolygonMap(0, 1, '-1:-1:-1:-1:-1:-1:-1:-1:-1:-1',
                                    '-1:-1:-1:-1:-1:-1:-1:-1:-1:-1');
                            } else {
                                playerApi.MsSetPolygonMap(1, 4, plgX, plgY);
                            }
                        }
                        if (this.regionalPeopleEnable[0] || this.regionalPeopleEnable[1] || this.regionalPeopleEnable[2] || this.regionalPeopleEnable[3]) {
                            this.drawTimerFlag = true;
                            this.drawTimer = setTimeout(() => {
                                this.update_regional_people();
                            }, 100);
                        }
                    }
            }
            if ((this.overlaySel >= 0 && this.overlaySel <= 4 || this.overlaySel == 7) && !this.trackEnable) {
                this.autoTrackTimerFlag = false;
                clearTimeout(this.autoTrackTimer);
            }
        },
        destroy_lpr_roi () {
            for (let i=0; i<this.LPR_ROI_NUM; i++) {
                playerApi.MsSetParam("clearlprroi", i);
                playerApi.MsSetParam("lprroidisableindex", i);
            }
            playerApi.MsSetParam("showlprroilines", "0");
        },
        set_plugin_lpr () {
            playerApi.MsSetParam("isLPR", "1");
            playerApi.MsSetParam("showlprroilines","0");
            let val = 1;
            if (this.lprWidth == 1920 && this.lprHeight == 1080)
                val = 0;
            else if (this.lprWidth == 1280 && this.lprHeight == 720)
                val = 1;
            else if (this.lprWidth == 640 && this.lprHeight == 360)
                val = 2;
            else if (this.lprWidth == 320 && this.lprHeight == 176)
                val = 3;
            switch (val) {
                case 0:
                    playerApi.MsSetParam("LPRResolutionW", "1920");
                    playerApi.MsSetParam("LPRResolutionH", "1080");
                    break;
                case 1:
                    playerApi.MsSetParam("LPRResolutionW", "1280");
                    playerApi.MsSetParam("LPRResolutionH", "720");
                    break;
                case 2:
                    playerApi.MsSetParam("LPRResolutionW", "640");
                    playerApi.MsSetParam("LPRResolutionH", "360");
                    break;
                case 3:
                    playerApi.MsSetParam("LPRResolutionW", "320");
                    playerApi.MsSetParam("LPRResolutionH", "176");
                    break;
            }
        },
        draw_lpr_roi () {
            playerApi.MsSetParam("drawmask","1");
            init_player_paint(6, 1);
            let region = '', i = 0, roiRegion = '';
            if (this.lprScene == 0)
                roiRegion = this.lprRoiRegion;
            else {
                if (this.curPresetPos >= 1 && this.curPresetPos <= 4) {
                    roiRegion = this.lprRoiRegionPreset[this.curPresetPos - 1];
                } else {
                    roiRegion = ['0:0:0:0', '0:0:0:0', '0:0:0:0', '0:0:0:0'];
                }
            }

            CheckBrowserType();
            this.set_plugin_lpr();
            if (browser_IE) {
                for (let i=0; i<this.LPR_ROI_NUM; i++) {
                    if (roiRegion[i] != '0:0:0:0') {
                        let roiArr = roiRegion[i].split(':');
                        let width = parseInt(roiArr[2]) - parseInt(roiArr[0]);
                        width = parseInt(width + 0.5);
                        let height = parseInt(roiArr[3]) - parseInt(roiArr[1]);
                        height = parseInt(height + 0.5);
                        let left = parseInt(roiArr[0]) * 1.0 / this.lprWidth * 100.0;
                        let top = parseInt(roiArr[1]) * 1.0 / this.lprHeight * 100.0;
                        let right = parseInt(roiArr[2]) * 1.0 / this.lprWidth * 100.0;
                        let bottom = parseInt(roiArr[3]) * 1.0 / this.lprHeight * 100.0;
                        let newRegion = `${parseInt(left)}:${parseInt(top)}:${parseInt(right)}:${parseInt(bottom)}`;
                        region += `${i}:${newRegion}:${width}:${height};`;
                    }
                }
                playerApi.MsSetParam("showlprroilines","1");
                playerApi.MsSetParam("lprroilines", region);
            } else {
                for (let i=0; i<this.LPR_ROI_NUM; i++) {
                    if (roiRegion[i] != '0:0:0:0')
                        region += `${i}:${roiRegion[i]};`
                }
                playerApi.MsSetParam("lprroirects2", region);
                playerApi.MsSetParam("nomovewnd","1");
            }
        },
        change_lpr_lays (val) {
            if (!this.isLprMode) {
                this.destroy_lpr_roi();
                return ;
            }
            this.lprSel = val;
            ASTGUI.cookies.setCookie("lprType", this.lprSel);
            if (this.lprSel == 0 || this.zoomEnable)
                this.destroy_lpr_roi();
            else if (this.lprSel == 1)
                this.draw_lpr_roi();
        },
        click_play () {
            if (this.isFisheye) {
                let url, ipcUrl = '&fishviewid=';
                if (this.curUrlListIndex != -1) {
                    if (this.stopAllFishVideo == 1) {
                        let base = this.get_fish_view_id(this.curUrlListIndex) - this.curUrlListIndex;
                        for (let j=base; j<base + this.channelPlay.length; j++) {
                            this.fishView[j] = -1;
                        }
                        this.stopAllFishVideo = 0;
                    }
                    if (this.isRecording)
                        this.click_record();

                    if (this.curWndPlay[this.curWndChannel] == 1 &&
                        this.playUrlIndex[this.curWndChannel] != this.curUrlListIndex) {
                        playerApi.MsDisConnectNew(this.curWndChannel);
                        let bPlayAudio = playerApi.MsGetParam("IsPlayAudio");
                        if (bPlayAudio == 0) {
                            this.playAudio = 0;
                        } else {
                            this.playAudio = 1;
                        }
                        let bTalk = playerApi.MsGetParam("IsDoTalk");
                        if (bTalk == 0 && this.playTalk == 1) {
                            this.playTalk = 0;
                            this.close_talk();
                        }
                        if (this.updateFishView == 1)
                            this.fishView[this.get_fish_view_id(this.playUrlIndex[this.curWndChannel])] = -1;
                    }

                    for (let i=0; i<this.playUrlIndex.length; i++) {
                        if (this.playUrlIndex[i] == this.curUrlListIndex) {
                            playerApi.MsDisConnectNew(i);
                            let bPlayAudio = playerApi.MsGetParam("IsPlayAudio");
                            if (bPlayAudio == 0)
                                this.playAudio = 0;
                            else
                                this.playAudio = 1;
                            let bTalk = playerApi.MsGetParam("IsDoTalk");
                            if (bTalk == 0 && this.playTalk == 1) {
                                this.playTalk = 0;
                                this.close_talk();
                            }

                            this.playUrlIndex[i] = -1;
                            if (this.updateFishView == 1)
                                this.fishView[this.get_fish_view_id(this.curUrlListIndex)] = -1;

                            if (i == this.curWndChannel && this.curWndPlay[this.curWndChannel] == 1) {
                                this.curWndPlay[this.curWndChannel] = 0;
                                this.change_channel_icon(this.curUrlListIndex, 0);
                                this.update_select_wnd_status(this.fishCurWnd);
                                if (this.updateFishView == 1) {
                                    let viewId = this.get_fish_view_id(this.curUrlListIndex);
                                    ipcUrl += viewId;
                                    ipcUrl += '&index=-1';
                                    this.fishView[viewId] = -1;
                                    let fishPos = '';
                                    for (let j=0; j<fishStreamType.FSH_MAX; j++) {
                                        fishPos += this.fishView[j] + ';';
                                    }
                                    ASTGUI.cookies.setLoginCookie("fishposition", fishPos);	//reserve 1000 days
                                }
                                this.isPlaying = 0;
                                this.set_fish_play_icon();
                                return ;
                            }
                            this.curWndPlay[i] = 0;
                            break;
                        }
                    }

                    url = GetCamPlayerUrl(this.curUrlListIndex);
                    this.curWndPlay[this.curWndChannel] = 1;
                    this.change_channel_icon(this.playUrlIndex[this.curWndChannel], 0);
                    this.playUrlIndex[this.curWndChannel] = this.curUrlListIndex;
                    console.log('MsConnectUrlNew', url, this.curWndChannel, this.curUrlListIndex);
                    playerApi.MsConnectUrlNew(url, 0, 1, 0, this.curWndChannel, this.curUrlListIndex);
                    let bPlayAudio = playerApi.MsGetParam("IsPlayAudio");
                    if (bPlayAudio == 0)
                        this.playAudio = 0;
                    else
                        this.playAudio = 1;
                    
                    if (this.curUrlListIndex == 0 || this.fishCorrect == fishCorrectMode.JOINT_VIDEO)
                        this.set_fish_channel_index(this.fishDisplay, this.curWndChannel);
                    if (this.curUrlListIndex == 1) {
                        this.set_panorama_channel_index(this.fishDisplay, this.curWndChannel);
                    }

                    this.set_fish_ptz_region(this.fishPtzRegion);
                    
                    this.curWndPlay[this.curWndChannel] = 1;
                    this.change_channel_icon(this.curUrlListIndex, 1);
                    this.update_select_wnd_status(this.fishCurWnd);

                    //record view position
                    if (this.updateFishView == 1) {
                        let viewId = this.get_fish_view_id(this.curUrlListIndex);
                        ipcUrl += viewId;
                        ipcUrl += `&index=${this.fishCurWnd}`;
                        this.fishView[viewId] = this.fishCurWnd;
                        let fishPos = '';
                        for (let j=0; j<fishStreamType.FSH_MAX; j++)
                            fishPos += this.fishView[j] + ';';
                        ASTGUI.cookies.setLoginCookie("fishposition", fishPos);	//reserve 1000 days
                    }

                    this.set_fish_play_icon();
                } else {
                    if (this.isPlaying == 0) {
                        this.stopAllFishVideo = 0;
                        this.switchFlag = 3;
                        if (Judge_Browser_Ver() === 0)
                            playerApi.MsGetPlayer().showStreamInfo = 2;
                        if (this.fishCorrect == fishCorrectMode.MUL_STREAM) {
                                this.window_switch(this.windowsModel);
                        } else {
                            this.set_correct_model(this.fishDisplay);
                            this.window_switch(-1);
                        }
                        this.isPlaying = 1;
                    } else {
                        if (Judge_Browser_Ver() === 0) {
                            let streamObj = document.getElementsByClassName('streamInfo')[0];
                            streamObj.style.display = 'none';
                            if (document.getElementsByClassName('selCanvas').length)
                                document.getElementsByClassName('selCanvas')[0].style.display = 'none';
                        }
                        this.close_all_stream_display();
                        this.fish_data_init();
                        this.close_talk();
                        for (let i=0; i<this.channelPlay.length; i++) {
                            this.change_channel_icon(i, 0);
                        }
                        if (this.enableAudio)
                            playerApi.MsCloseCall();
                        if (this.isRecording)
                            this.click_record();
                        this.isPlaying = 0;
                        this.update_select_wnd_status(0);
                        this.stopAllFishVideo = 1;
                    }
                }
            } else {
                let canvasObj = document.querySelectorAll('.selCanvas')
                if (this.isPlaying == 0) {
                    PlayActiveXM(this.streamSel, this.get_smart_stream(this.streamSel),
                        this.update_select_wnd_status, this.get_fish_mouse_move, this.set_correct_model, this);

                    this.isPlaying = 1;
                    if (Judge_Browser_Ver() === 0) {
                        let streamObj = document.getElementsByClassName('streamInfo')[0];
                        streamObj.classList = [];
                        streamObj.classList.add('streamInfo', 'posRight');
                        streamObj.style.display = '';
                        playerApi.MsGetPlayer().m_Players[0].showStreamInfo = 2;
                    }

                    this.playAudio = localStorage.getItem("IsPlayAudio") == "true" ? 1 : 0;
                    canvasObj.forEach(item=>{
                        item.style.display = ''
                    })
                    // document.getElementsByClassName('selCanvas')[0].style.display = '';
                } else {
                    if (this.isRecording)
                        this.click_record();
                    if (this.zoomEnable) {
                        this.zoomEnable = 0;
                        playerApi.MsSetParam("zoomenable", "0");
                    }

                    playerApi.MsDisConnect();
                    playerApi.MsStopTalking();
                    if (this.playAudio)
                        playerApi.MsCloseCall();
                    this.isPlaying = 0;
                    canvasObj.forEach(item=>{
                        item.style.display = 'none'
                    })
                    document.querySelector('.streamInfo')&&(document.querySelector('.streamInfo').style.display = 'none')
                    // document.getElementsByClassName('selCanvas')[0].style.display = 'none';
                }
            }
        },
        click_audio (flag) {
            if (this.sysInfo.isfisheye == '1') {
                let bWndPlay = 0;
                for (let i=0; i<this.curWndPlay.length; i++) {
                    if (this.curWndPlay[i] == 1) {
                        bWndPlay = 1;
                        break;
                    }
                }
                if (this.playAudio || flag) {
                    bWndPlay = 1;
                }
                if (bWndPlay == 0) {
                    this.playAudio = 0;
                    return ;
                }
            }
            this.playAudio = ! this.playAudio;
            playerApi.MsSetParam("PlayAudio", this.playAudio ? '1' : '0');
            localStorage.setItem("IsPlayAudio",this.playAudio);
            liveview_enable_play_audio(this.playAudio);
        },
        close_talk () {
            playerApi.MsStopTalking();
        },
        click_talk () {
            if (this.sysInfo.isfisheye == '1') {
                let bWndPlay = 0;
                for (let i=0; i<this.curWndPlay.length; i++) {
                    if (this.curWndPlay[i] == 1) {
                        bWndPlay = 1;
                        break;
                    }
                }
                if (bWndPlay == 0) {
                    this.playTalk = 0;
                    playerApi.MsCloseCall();
                    return ;
                }
            }
            let localip = GetIpAddr();
            if(localip.indexOf("[") != -1 && localip.indexOf("]") != -1) {
                localip = localip.substr(1,localip.length-2);
            }
            let localport = window.location.port;
            let protocolStr = document.location.protocol;
            if (localport == "") {
                localport = 80;
            } else {
                localport = parseInt(localport);
            }
            this.playTalk = !this.playTalk;
            if(this.playTalk) {
                if(protocolStr.indexOf("https") >= 0) {
                    playerApi.MsStartToTalkHttps(localip, this.httpsPort,
                        ASTGUI.cookies.getCookie("loginname"), window.localStorage.getItem('password'));
                } else {
                    playerApi.MsStartToTalk(localip, localport,
                        ASTGUI.cookies.getCookie("loginname"), window.localStorage.getItem('password'));
                    if (!this.isIE) {
                        this.playTalk = !this.playTalk;
                    }
                }
            } else {
                playerApi.MsStopTalking();
            }
        },
        click_cut () {
            playerApi.MsCaptureImg(2);
        },
        click_record () {
            if (this.sysInfo.isfisheye == '1' && this.fishCorrect == fishCorrectMode.MUL_STREAM) {
                if (this.isRecordingFish[this.fishCurWnd] == 1)
                    this.isRecording = 1;
                else
                    this.isRecording = 0;
                playerApi.MsRecord(this.isRecording);
                if (this.isRecording == 0) {
                    this.isRecording = 1;
                    this.isRecordingFish[this.fishCurWnd] = 1;
                } else {
                    this.isRecording = 0;
                    this.isRecordingFish[this.fishCurWnd] = 0;
                }
            } else {
                playerApi.MsSetParam('recordtime', this.recordTime);
                playerApi.MsRecord(this.isRecording);
                this.isRecording = 1 - this.isRecording;
            }
        },
        click_zoom () {
            if (this.isPlaying == 0)
                return;
            if ((this.sysInfo.isspeed == '1' || this.sysInfo.isptzbullet == '1' || this.sysInfo.isminiptzdome == '1')  &&
                this.posEnable && !is_anony()) {
                tipsBox(this.$t('zoomWithout3d'), 'warning');
                return;
            }
            if (this.sysInfo.isfisheye == '1' && this.fishCorrect == fishCorrectMode.MUL_STREAM) {
                this.zoomEnable = 1 - this.curWndZoom[this.fishCurWnd];
                playerApi.MsSetParam('zoomenable', this.zoomEnable);
                this.curWndZoom[this.fishCurWnd] = this.zoomEnable;
            } else {
                this.zoomEnable = 1 - this.zoomEnable;
                playerApi.MsSetParam('zoomenable', this.zoomEnable);
            }
            if (this.lprSel == 1) {
                if (this.zoomEnable) {
                    this.destroy_lpr_roi();
                } else {
                    this.draw_lpr_roi();
                }
            }
        },
        hide_all_lines () {
            this.hide_line(1);
            this.hide_line(2);
            this.hide_line(3);
            this.hide_line(4);
        },
        hide_line (id) {
            playerApi.MsSetLineID(id);
            playerApi.MsSetIRLineRegion(1,'0;0;0;0');
        },
        show_all_lines () {
            this.show_line(1,0);
            this.show_line(2,0);
            this.show_line(3,0);
            this.show_line(4,0);
        },
        show_line (id, color) {
            if (this.sysInfo.isfisheye == '1') {
                if (!this.fish_support_vca(this.fishCorrect, this.fishDisplay, this.fishInstall))
                    return;
            } else if (this.sysInfo.isusingcropvideo == '1' && this.streamSel != 0)
                return ;
            if (id >= 5)
                playerApi.MsSetLineID(id - 4);
            else
                playerApi.MsSetLineID(id);
            playerApi.MsSetIRLineColor(color);
            this.draw_line_by_type(id);
        },
        draw_line_by_type (id) {
            let dir, coutLine;
            let curPreset = this.vcaCurPreset;
            if (id >= 5) {
                if (curPreset >= 1 && curPreset <= 4 && this.countlinescene) {
                    curPreset = curPreset - 1;
                    dir = this.countLineDirPreset[curPreset][id - 5];
                    coutLine = this.countLinePreset[curPreset][id - 5];
                    playerApi.MsSetIRLineDir(1, dir);
                    if (coutLine != '1;2;3;4') {
                        playerApi.MsSetIRLineRegion(1, coutLine);
                    } else {
                        playerApi.MsSetIRLineRegion(1, '0;0;0;0');
                    }
                } else {
                    dir = this.countLineDir[id - 5];
                    coutLine = this.countLine[id - 5];
                    if (this.countlinescene) {
                        playerApi.MsSetIRLineRegion(1, '0;0;0;0');
                    } else {
                        playerApi.MsSetIRLineDir(1, dir);
                        if (coutLine != '1;2;3;4') {
                            playerApi.MsSetIRLineRegion(1, coutLine);
                        } else {
                            playerApi.MsSetIRLineRegion(1, '0;0;0;0');
                        }                   
                    }
                }
            } else {
                if (curPreset >= 1 && curPreset <= 4 && this.linecrossingscene) {
                    curPreset = curPreset - 1;
                    dir = this.crossLineDirPreset[curPreset][id - 1];
                    coutLine = this.crossLinePreset[curPreset][id - 1];
                    playerApi.MsSetIRLineDir(1, dir);
                    if (coutLine != '1;2;3;4') {
                        playerApi.MsSetIRLineRegion(1, coutLine);
                    } else {
                        playerApi.MsSetIRLineRegion(1, '0;0;0;0');
                    }
                } else {
                    dir = this.crossLineDir[id - 1];
                    coutLine = this.crossLine[id - 1];
                    if (this.linecrossingscene) {
                        playerApi.MsSetIRLineRegion(1, '0;0;0;0');
                    } else {
                        playerApi.MsSetIRLineDir(1, dir);
                        if (coutLine != '1;2;3;4') {
                            playerApi.MsSetIRLineRegion(1, coutLine);
                        } else {
                            playerApi.MsSetIRLineRegion(1, '0;0;0;0');
                        }                   
                    }
                }
            }

            /*
            if (curPreset >= 1 && curPreset <= 4) {
                curPreset = curPreset - 1;
                if (id >= 5) {
                    dir = this.countLineDirPreset[curPreset][id - 5];
                    coutLine = this.countLinePreset[curPreset][id - 5];
                } else {
                    dir = this.crossLineDirPreset[curPreset][id - 1];
                    coutLine = this.crossLinePreset[curPreset][id - 1];
                }
                playerApi.MsSetIRLineDir(1, dir);
                if (coutLine != '1;2;3;4') {
                    playerApi.MsSetIRLineRegion(1, coutLine);
                } else {
                    playerApi.MsSetIRLineRegion(1, '0;0;0;0');
                }
            } else {
                if (id >= 5) {
                    dir = this.countLineDir[id - 5];
                    coutLine = this.countLine[id - 5];
                } else {
                    dir = this.crossLineDir[id - 1];
                    coutLine = this.crossLine[id - 1];
                }
                if (((id >= 5) && this.linecrossingscene == 1) || ((id < 5)) && (this.countlinescene == 1)) {
                    playerApi.MsSetIRLineRegion(1, '0;0;0;0');
                } else {
                    playerApi.MsSetIRLineDir(1, dir);
                    if (coutLine != '1;2;3;4') {
                        playerApi.MsSetIRLineRegion(1, coutLine);
                    } else {
                        playerApi.MsSetIRLineRegion(1, '0;0;0;0');
                    }                   
                }
            }
            */
        },
        show_auto_track () {
            if (this.sysInfo.isfisheye == '1') {
                if (!this.fish_support_show_track())
                    return;
            }
            if (this.sysInfo.isusingcropvideo == '1') {
                if (!this.autoTrackTimerFlag) {
                    this.autoTrackTimerFlag = true;
                    this.check_track();
                }
            } else {
                if (this.trackEnable && this.showTrack) {
                    if (!this.autoTrackTimerFlag) {
                        this.autoTrackTimerFlag = true;
                        this.check_track();
                    }
                }
            }
        },
        clear_auto_track () {
            this.autoTrackTimerFlag = false;
            clearTimeout(this.autoTrackTimer);
            if (xPlayer != null)
                playerApi.MsSetPersonRectRegion(0, '0:0:0:0:0:0');	//for webplugin
            else
                playerApi.MsSetPersonRectRegion(1, '0:0:0:0:0:0');
        },
        draw_liveview_rect (type, data) {
            if(type == 0)
                this.vcaRect = data;
            else if(type == 1)
                this.trackRect = data;
            playerApi.MsSetPersonRectRegion(1, this.vcaRect + this.trackRect);
        },
        fish_support_vca (correct, display, install) {
            if (this.sysInfo.isfisheye == '1') {
                if (display == fishDisplayMode.DISPLAY_1O)
                    return 1;
                else if (install == fishInstallMode.WALL && display == fishDisplayMode.DISPLAY_360P)
                    return 0;       // cv22 not support
                else if (correct == fishCorrectMode.MUL_STREAM && (display == fishDisplayMode.DISPLAY_1O3R ||
                    display == fishDisplayMode.DISPLAY_1O1P3R))
                    return 1;
                else if (correct == fishCorrectMode.MUL_STREAM && install == fishInstallMode.WALL &&
                    display == fishDisplayMode.DISPLAY_1P3R)
                    return 0;       // cv22 not support
                else
                    return 0;
            }
            return 0;
        },
        fish_support_show_track() {
            if (playerApi.MsGetParam("fisheyeCorrectionModel") == '1')
                return 0;
            if (this.fishInstall == fishInstallMode.CEILING && (this.fishDisplay >= fishDisplayMode.DISPLAY_4R &&
                this.fishDisplay <= fishDisplayMode.DISPLAY_1O1P3R))
                return 1;
            else
                return 0;
        },
        fish_support_auto_track (install, display) {
            if (playerApi.MsGetParam("fisheyeCorrectionModel") == '1')
                return 0;
            if (install == fishInstallMode.CEILING && (display >= fishDisplayMode.DISPLAY_4R &&
                display <= fishDisplayMode.DISPLAY_1O1P3R))
                return 1;
            else
                return 0;
        },
        check_track() {
            let url = '&trackrgn';
            send_vb_htm(url).then( res => {
                let txt = res;
                let ix = txt.indexOf('OK trackrgn=');
                if (ix >= 0) {
                    let code = txt.substring(ix +12);
                    if (this.sysInfo.isfisheye == '1') {
                        playerApi.MsSetObjectTrailEnable(0);
                    } else
                        this.draw_liveview_rect(1, code);
                }
                if (this.sysInfo.isfisheye == '1' && this.fish_support_show_track()) {
                    this.clear_auto_track();
                    return;
                }
                if (this.sysInfo.isusingcropvideo == '1') {
                    if (this.streamSel != 0) {
                        this.clear_auto_track();
                        return;
                    }
                    if (this.overlaySel == 9 && this.autoTrackTimerFlag) {
                        this.autoTrackTimer = setTimeout( () => {
                            this.check_track();
                        }, 300);
                    }
                    return ;
                }
            });
            if (this.autoTrackTimerFlag) {
                this.autoTrackTimer = setTimeout( () => {
                    this.check_track()
                }, 300);
            }
        },
        async update_person (index) {
            let url = `&personpoint=${index}`;
            let txt = await send_vb_htm(url);
            let ix = txt.indexOf('OK personpoint=');
            if (ix >= 0) {
                let code = txt.substring(ix + 15);
                this.draw_liveview_rect(0, code);
            }
            
            if (this.hasDestroy) {  // clear before quit
                playerApi.MsSetPersonRectRegion(1, "0:0:0:0:0:0");
                playerApi.MsSetPersonRectWithTime(1, "");
                playerApi.MsSetRegionAlarm("0:0:-1:-1:-1:-1");
                return;
            }
            if (this.drawTimerFlag) {
                if (this.overlaySel == this.find_overlays_by_index(index)) {
                    this.drawTimer = setTimeout(() => {
                        this.update_person(index);
                    }, 100);
                }
            }
        },
        update_count_line_data () {
            let url = '&peoplecountdata';
            send_vb_htm(url).then( res => {
                let txt = res;
                let ix = txt.indexOf('OK peoplecountdata=');
                if (this.overlaySel == 8) {
                    if (ix >= 0) {
                        let code = txt.substring(ix + 19);
                        playerApi.MsSetCoutLineData(code);
                        this.show_line(5, this.countLineAlarm[0]);
                        this.show_line(6, this.countLineAlarm[1]);
                        this.show_line(7, this.countLineAlarm[2]);
                        this.show_line(8, this.countLineAlarm[3]);
                    }
                    if (this.drawDataTimerFlag) {
                        this.drawDataTimer = setTimeout( () => {
                            this.update_count_line_data();
                        }, 100);
                    }
                } else {
                    playerApi.MsSetCoutLineData("-1:-1:-1:-1;-1:-1:-1:-1;-1:-1:-1:-1;-1:-1:-1:-1;");
                }
            })
        },
        update_regional_people () {
            let url = '&regionalpeople';
            send_vb_htm(url).then( res => {
                let txt = res;
                let ix = txt.indexOf('OK regionalpeople=');
                let region = txt.indexOf('&regionalarm=');
                if (ix >= 0 && this.overlaySel == this.find_overlays_by_index(9)) {
                    let code = txt.substring(ix + 18, region);
                    let code1 = txt.substring(region + 13);
                    playerApi.MsSetPersonRectWithTime(1, code);
                    playerApi.MsSetRegionAlarm(code1);
                }
                if (this.drawTimerFlag && this.overlaySel == 10) {
                    this.drawTimer = setTimeout(() => {
                        this.update_regional_people();
                    }, 100);
                } else {
                    playerApi.MsSetPersonRectWithTime(1, "");
                    playerApi.MsSetRegionAlarm("0:0:-1:-1:-1:-1");
                }
            })
        },
        find_overlays_by_index (index) {
            var overlays = -1;
            switch(index) {
                case 0:
                    overlays = 1;
                    break;
                case 1:
                    overlays = 2;
                    break;
                case 2:
                    overlays = 3;
                    break;
                case 4:
                    overlays = 6;
                    break;
                case 5:
                    overlays = 4;
                    break;
                case 6:
                    overlays = 10;
                    break;
                case 7:
                    overlays = 8;
                    break;
                case 9:
                    overlays = 10;
                    break;
            }
            return overlays;
        },
        async update_object_track() {
            let url = 'vcaleftpoint';
            let txt = await send_vb_htm(url);
            let ix = txt.indexOf('OK vcaleftpoint=');
            if (ix >= 0) {
                let code = txt.substring(ix + 16);
                this.draw_liveview_rect(0, code);
            }
            if (this.drawTimerFlag) {
                this.drawTimer = setTimeout( () => {
                    this.update_object_track();
                }, 500);
            }
        },
        init_stream_options() {
            this.streamOptions = [{ value: 0, label: this.$t("primaryStream") }];
            if (this.subStreamEnable)
                this.streamOptions.push({ value: 1, label: this.$t("subStream") });
            if (this.triStreamEnable)
                this.streamOptions.push({ value: 2, label: this.$t("thirdStream") });
            if (this.fourStreamEnable)
                this.streamOptions.push({ value: 3, label: this.$t("fourthStream") });
            if (this.fifStreamEnable)
                this.streamOptions.push({ value: 4, label: this.$t("fifthStream") });
        },
        change_zoom_pos () {
            send_ptz_cmd_to_device_str(`zoompos=${this.zoomPos}`);
            // 同步79。3s内不能操作focus
            this.disableFocus = true
            if (this.disableFocusTimer) {
                clearTimeout(this.disableFocusTimer)
            }
            this.disableFocusTimer = setTimeout(() => {
                this.disableFocus = false
            }, 3000)
        },
        change_bright () {
            let url = `&brightness=${this.bright * 255 / 100 + 0.5}`;
            set_page_data('image', url);
        },
        change_contrast () {
            let url = `&contrast=${this.contrast * 255 / 100 + 0.5}`;
            set_page_data('image', url);
        },
        change_saturation () {
            let url = `&saturation=${this.saturation * 255 / 100 + 0.5}`;
            set_page_data('image', url);
        },
        change_sharp () {
            let url = `&sharpness=${this.sharp * 255 / 100 + 0.5}`;
            set_page_data('image', url);
        },
        change_dnr2d () {
            let url = `&dnr2level=${this.dnr2d * 255 / 100 + 0.5}`;
            set_page_data('image', url);
        },
        change_nflevel () {
            let url = `&nflevel=${this.nflevel * 255 / 100 + 0.5}`;
            set_page_data('image', url);
        },
        click_default () {
            this.bright = 50;
            this.contrast = 50;
            this.saturation = 50;
            this.sharp = 50;
            this.dnr2d = 50;
            this.nflevel = 50;
            let url = '&brightness=128&contrast=128&saturation=128&sharpness=128&dnr2level=128&nflevel=128';
            set_page_data('image', url);
        },
        display_switch (type) {
            console.log('display_switch', type);
            this.curDisplay = type;
            this.playTalk = 0;
        },
        close_all_stream_display () {
            for (let i=0; i<this.curWndPlay.length; i++) {
                if (this.curWndPlay[i] == 1) {
                    playerApi.MsDisConnectNew(i);
                    this.curWndPlay[i] = 0;
                }
            }
            playerApi.MsSetParam("allStop",'true');
            this.playAudio = 0;
            localStorage.setItem('IsPlayAudio', this.playAudio);
            this.playTalk = 0;
        },
        get_playing_channel_num () {
            let c = 0;
            for (let i=0; i<this.channelPlay.length; i++) {
                if (this.channelPlay[i] == 1)
                    c++;
            }
            return c;
        },
        set_correct_model (display) {
            if (this.fishCorrect == fishCorrectMode.JOINT_VIDEO && (this.fishDisplay == fishDisplayMode.DISPLAY_4R ||
                this.fishDisplay == fishDisplayMode.DISPLAY_1O3R || this.fishDisplay == fishDisplayMode.DISPLAY_1P3R))
                playerApi.MsSetParam("fishvideocorrectmodel", 1);
            else
                playerApi.MsSetParam("fishvideocorrectmodel", 0);
            playerApi.MsSetParam("livevideoui","1");
        },
        fish_data_init () {
            if (this.isRecording)
                this.click_record();
            for (let i=0; i<this.curWndPlay.length; i++) {
                this.recordStartFish[i] = 0;
                this.curWndZoom[i] = 0;
                this.curWndPlay[i] = 0;
                this.playUrlIndex[i] = -1;
            }
        },
        get_url_count (display) {
            if (!this.isFisheye || display == null || display == undefined)
                return 1;
            switch (display) {
                case fishDisplayMode.DISPLAY_1O:
                case fishDisplayMode.DISPLAY_360P:
                case fishDisplayMode.DISPLAY_180P:
                    return 1;
                case fishDisplayMode.DISPLAY_4R:
                case fishDisplayMode.DISPLAY_1O3R:
                    if (this.fishCorrect == fishCorrectMode.MUL_STREAM)
                        return 4;
                    else
                        return 1;
                case fishDisplayMode.DISPLAY_1P3R:
                    return 4;
                case fishDisplayMode.DISPLAY_1O1P3R:
                    return 5;
            }
        },
        build_channel_list (cnt) {
            this.channelPlay = [];
            for (let i=0; i<cnt; i++) {
                this.channelPlay.push(1);
            }
        },
        change_channel_icon (stream, flag) {
            if (stream < 0)
                return ;
            if (this.channelPlay.length == 1) {
                if (flag == 0)
                    this.channelPlay[0] = 0;
                else
                    this.channelPlay[0] = stream + 1;
            } else {
                this.channelPlay[stream] = flag;
            }
            this.channelPlay = [...this.channelPlay];
        },
        get_window_type () {
            let type = 0;
            switch (this.fishDisplay) {
                case fishDisplayMode.DISPLAY_1O:
                case fishDisplayMode.DISPLAY_360P:
                case fishDisplayMode.DISPLAY_180P:
                    type = 0;
                    break;
                case fishDisplayMode.DISPLAY_4R:
                case fishDisplayMode.DISPLAY_1O3R:
                case fishDisplayMode.DISPLAY_1P3R:
                    if (this.fishCorrect == fishCorrectMode.MUL_STREAM)
                        type = 1;
                    else
                        type = 0;
                    break;
                case fishDisplayMode.DISPLAY_1O1P3R:
                    type = 2;
                    break;
                default:
                    type = 1;
                    break;
            }
            return type;
        },
        get_win_by_channel (channel) {
            if (channel == 0)
                return 1;
            else if (channel == 1)
                return 2;
            else if (channel == 2)
                return 5;
            else
                return 3;
        },
        set_wnd_style (channel) {
            let wnd = 1;
            if (channel < 0 || channel > 6)
                return ;
            wnd = this.get_win_by_channel(channel);

            if (ASTGUI.cookies.getCookie("fisheyeCorrectionModel") == '1')
                playerApi.MsSetParam("fisheyeWndStyle", 1);
            else
                playerApi.MsSetParam("wndnums", wnd);
        },
        set_channel_record (state) {
            this.isRecording = state;
        },
        set_channel_zoom (state) {
            this.zoomEnable = state;
        },
        set_cur_channel (channel) {
            this.curWndChannel = channel;
            if (this.fishCorrect == fishCorrectMode.MUL_STREAM) {
                this.set_channel_record(this.recordStartFish[this.curWndChannel]);
                this.set_channel_zoom(this.curWndZoom[channel]);

                if (this.curWndZoom[this.curWndChannel] == 0)
                    this.zoomEnable = 0;
                else
                    this.zoomEnable = 1;
                if (this.recordStartFish[this.curWndChannel] == 0)
                    this.isRecording = 0;
                else
                    this.isRecording = 1;
            }
        },
        change_preset_data (display, sel) {
            if (sel < 0 || sel > 4) {
                this.displayDisablePtz = true;
                this.init_preset();
                return ;
            }
            if (display == fishDisplayMode.DISPLAY_1O || display == fishDisplayMode.DISPLAY_360P ||
                display == fishDisplayMode.DISPLAY_180P) {
                this.displayDisablePtz = true;
                this.init_preset();
                return ;
            }

            if ((display == fishDisplayMode.DISPLAY_1O3R || display == fishDisplayMode.DISPLAY_1P3R) && sel == 0) {
                this.displayDisablePtz = true;
                this.init_preset();
            } else if (display == fishDisplayMode.DISPLAY_1O1P3R && (sel == 0 || sel == 1)) {
                this.displayDisablePtz = true;
                this.init_preset();
            } else {
                this.displayDisablePtz = false;
                this.init_fish_preset(sel);
            }
        },
        change_tour_data (display, sel) {
            if (sel < 0 || sel > 4) {
                this.displayDisablePtz = true;
                this.init_tour();
                return ;
            }
            if (display == fishDisplayMode.DISPLAY_1O || display == fishDisplayMode.DISPLAY_360P ||
                display == fishDisplayMode.DISPLAY_180P) {
                this.displayDisablePtz = true;
                this.init_tour();
                return ;
            }

            if ((display == fishDisplayMode.DISPLAY_1O3R || display == fishDisplayMode.DISPLAY_1P3R) && sel == 0) {
                this.displayDisablePtz = true;
                this.init_tour();
            } else if (display == fishDisplayMode.DISPLAY_1O1P3R && (sel == 0 || sel == 1)) {
                this.displayDisablePtz = true;
                this.init_tour();
            } else {
                this.displayDisablePtz = false;
                this.tourArr = [...this.fishTourList[sel]];
                this.playTour = this.fishPlayTour[sel];
                this.setTour = -1;
            }
        },
        set_fish_ptz_region (region) {
            playerApi.MsSetParam("fishptzregion", region);
            return 0;
        },
        async get_fish_ptz_region () {
            let url = '&getfishptzregion';
            let txt = await send_vb_htm(url);
            let ix = txt.indexOf('OK getfishptzregion=');
            if (ix >= 0) {
                let code = txt.substring(ix + 20);
                this.fishPtzRegion = code;
                this.set_fish_ptz_region(this.fishPtzRegion);
            }
            if (this.bPtzTimer == 1) {
                this.ptzTimer = setTimeout( () => {
                    this.get_fish_ptz_region();
                }, 100);
            }
        },
        update_digital_label (start, operate) {
            this.bPtzTimer = 0;
            clearTimeout(this.ptzTimer);
            this.ptzTimer = setTimeout( () => {
                this.get_fish_ptz_region();
            }, 1000);
            playerApi.MsSetParam("ptzoperation", 0);
            if (start == 1) {
                if (operate == 1)
                    playerApi.MsSetParam("ptzoperation", 0);
                if (!this.bPtzTimer) {
                    this.bPtzTimer = 1;
                    this.ptzTimer = setTimeout( () => {
                        this.get_fish_ptz_region();
                    }, 100);
                }
            }
        },
        async update_fish_tour_play_and_auto_scan () {
            let stream = this.get_channel_index();
            if (stream < 0 || stream > 5)
                return ;
            let url = 'paratest=fishtourplaystatus.0&paratest=fishtourplaystatus.1&paratest=fishtourplaystatus.2' +
                '&paratest=fishtourplaystatus.3&paratest=fishtourplaystatus.4&paratest=fishautoscan.0' +
                '&paratest=fishautoscan.1&paratest=fishautoscan.2&paratest=fishautoscan.3&paratest=fishautoscan.4';
            let txt = await send_vb_htm(url);
                let tourFlag = 0;
                let ix = txt.indexOf(`OK fishtourplaystatus.${stream}`);
                if (ix >= 0) {
                    let code = txt.substring(ix + 24);
                    let tourPlayArr = code.split(';');

                    for (let i=0; i<this.PATROL_NUM; i++) {
                        if (tourPlayArr[i] == '1') {
                            this.playTour = this.fishPlayTour[stream] = i + 1;
                            tourFlag = 1;
                        }
                    }
                    if (!tourFlag)
                        this.playTour = 0;
                }
                if (stream == 0 && (this.fishDisplay == fishDisplayMode.DISPLAY_1O ||
                    this.fishDisplay == fishDisplayMode.DISPLAY_1O1P3R)) {
                    for (let i=0; i<this.FISH_MAX_VIEW_NUM; i++) {
                        ix = txt.indexOf(`OK fishtourplaystatus.${i}`);
                        if (ix >= 0) {
                            let code = txt.substring(ix + 24);
                            let tourPlayArr = code.split(';');

                            for (let i=0; i<this.PATROL_NUM; i++) {
                                if (tourPlayArr[i] == '1') {
                                    tourFlag = 1;
                                    break;
                                }
                            }
                        }
                    }
                }
                if (tourFlag == 1)
                    this.update_digital_label(1);

                let autoFlag = 0;
                ix = txt.indexOf(`OK fishautoscan.${stream}`);
                if (ix > 0) {
                    if (this.is_fish_software_correct()) {
                        if (playerApi.MsGetParam("fisheyeCorrectionCurCmd") == '13') {
                            this.autoScanStatus = 1;
                        } else {
                            this.autoScanStatus = 0;
                        }
                        let tourId = parseInt(playerApi.MsGetParam("fisheyeCorrectionCurTour"));
                        for (let i=0; i<this.PATROL_NUM; i++) {
                            if (i == tourId) {
                                this.playTour = this.fishPlayTour[stream] = i + 1;
                            }
                        }
                    } else {
                        let state = parseInt(txt.charAt(ix + 18));
                        this.autoScanStatus = state;
                        this.fishAutoScan[stream] = state;
                    }
                    for (let i=0; i<this.FISH_MAX_VIEW_NUM; i++) {
                        if (this.fishAutoScan[i] == 1) {
                            autoFlag = 1;
                            break;
                        }
                    }
                }
                if (stream == 0 && (this.fishDisplay == fishDisplayMode.DISPLAY_1O3R ||
                    this.fishDisplay == fishDisplayMode.DISPLAY_1O1P3R)) {
                    for (let j=0; j<this.FISH_MAX_VIEW_NUM; j++) {
                        ix = txt.indexOf(`OK fishautoscan${j}`);
                        if (ix >= 0) {
                            let state = parseInt(txt.charAt(ix + 17));
                            this.fishAutoScan[j] = state;
                            for (let i=0; i<this.FISH_MAX_VIEW_NUM; i++) {
                                if (this.fishAutoScan[i] == 1) {
                                    autoFlag = 1;
                                    break;
                                }
                            }
                        }
                    }
                }
                if (autoFlag == 1)
                    this.update_digital_label(1);
                if (autoFlag == 0 && tourFlag == 0) {
                    clearTimeout(this.ptzTimer);
                    this.bPtzTimer = 0;
                }
            if (this.fishTourTimerFlag) {
                this.fishTourTimer = setTimeout( () => {
                    this.update_fish_tour_play_and_auto_scan();
                }, 1000);
            }
        },
        modify_preset_by_display (display, sel) {
            this.change_preset_data(display, sel);
        },
        modify_tour_by_display (display, sel) {
            this.change_tour_data(display, sel);
            if (!this.fishTourTimerFlag) {
                this.fishTourTimerFlag = true;
                this.update_fish_tour_play_and_auto_scan();
            }
        },
        update_auto_scan () {
            if (this.is_fish_software_correct()) {
                if (playerApi.MsGetParam("fisheyeCorrectionCurCmd") == '13')
                    this.autoScanStatus = 1;
                else
                    this.autoScanStatus = 0;
                let tourId = parseInt(playerApi.MsGetParam("fisheyeCorrectionCurTour"));
                this.playTour = this.fishPlayTour[this.get_channel_index()] = tourId;
                let ptz = playerApi.MsGetParam("FisheyeSoftwareCorrectionPtzRegion");
                if (ptz == '') {
                    this.modify_preset_by_display(fishDisplayMode.DISPLAY_1O3R, 0);
                    this.modify_tour_by_display(fishDisplayMode.DISPLAY_1O3R, 0);
                } else {
                    this.modify_preset_by_display(fishDisplayMode.DISPLAY_1O3R, 1);
                    this.modify_tour_by_display(fishDisplayMode.DISPLAY_1O3R, 1);
                }
                return ;
            }

            let index = this.get_channel_index();
            if (index < 0)
                this.autoScanStatus = 0;
            else {
                if (this.fishAutoScan[index] == 0)
                    this.autoScanStatus = 0;
                else
                    this.autoScanStatus = 1;
            }
        },
        update_preset_and_tour () {
            if (this.lastStream == this.get_channel_index())
                return ;
            else
                this.lastStream = this.get_channel_index();
            let display = this.fishDisplay;
            if (this.is_fish_software_correct())
                display = fishDisplayMode.DISPLAY_1O3R;
            this.modify_preset_by_display(display, this.get_channel_index());
            this.modify_tour_by_display(display, this.get_channel_index());
        },
        update_select_wnd_status (wnd) {
            if (this.get_channel_index() < 0)
                this.lastStream = -2;
            this.fishCurWnd = wnd;
            if (this.fishCurWnd < 0) {
                this.fishCurWnd = -1;
                try {
                    let player = playerApi.MsGetPlayer();
                    player.SetCurWnd(this.fishCurWnd);
                } catch (e) {
                } finally {
                    this.set_cur_channel(-1);
                }
            } else {
                try {
                    if (this.get_channel_index() >= 0 && ((this.fishDisplay == fishDisplayMode.DISPLAY_1P3R ||
                        this.fishDisplay == fishDisplayMode.DISPLAY_1O1P3R) ||
                        (this.fishCorrect == fishCorrectMode.MUL_STREAM &&
                        (this.fishDisplay == fishDisplayMode.DISPLAY_4R ||
                        this.fishDisplay == fishDisplayMode.DISPLAY_1O3R ||
                        this.fishDisplay == fishDisplayMode.DISPLAY_1P3R))))
                        this.selectedChannel = this.get_channel_index();
                    else
                        this.selectedChannel = this.fishCurWnd;
                } catch (e) {

                } finally {
                    this.update_auto_scan();
                    this.set_cur_channel(this.fishCurWnd);
                    if (this.updatePreset == 1 && this.lastStream != this.get_channel_index()) {
                        this.update_preset_and_tour();
                    }
                    if (this.updatePreset == 1 && (this.fishDisplay == fishDisplayMode.DISPLAY_1O3R ||
                        this.fishDisplay == fishDisplayMode.DISPLAY_1O1P3R)) {
                        if (!this.bPtzTimer) {
                            this.ptzTimer = setTimeout( () => {
                                this.get_fish_ptz_region();
                            }, 200);
                        }
                    }
                }
            }
        },
        get_fish_view_id (stream) {
            let base = 0;
            switch (this.fishDisplay) {
                case fishDisplayMode.DISPLAY_1O:
                    base = fishStreamType.FSH_1O_MAIN;
                    break;
                case fishDisplayMode.DISPLAY_360P:
                    base = fishStreamType.FSH_1P_MAIN;
                    break;
                case fishDisplayMode.DISPLAY_180P:
                    base = fishStreamType.FSH_2P_MAIN;
                    break;
                case fishDisplayMode.DISPLAY_4R:
                    base = fishStreamType.FSH_4R_MAIN;
                    break;
                case fishDisplayMode.DISPLAY_1O3R:
                    base = fishStreamType.FSH_1O3R_MAIN;
                    break;
                case fishDisplayMode.DISPLAY_1P3R:
                    base = fishStreamType.FSH_1P3R_MAIN;
                    break;
                case fishDisplayMode.DISPLAY_1O1P3R:
                    base = fishStreamType.FSH_1O1P3R_MAIN;
                    break;
                default:
                    break;
            }
            return base + stream;
        },
        set_fish_play_icon () {
            let play = 0;
            for (let i=0; i<this.playUrlIndex.length; i++) {
                if (this.playUrlIndex[i] != -1) {
                    play = 1;
                    break;
                }
            }
            if (play == 1)
                this.isPlaying = 1;
            else
                this.isPlaying = 0;
            if (this.zoomEnable == 1) {
                this.zoomEnable = 0;
                playerApi.MsSetParam("zoomenable", "0");
                this.curWndZoom[this.fishCurWnd] = this.zoomEnable;
            }
        },
        set_fish_channel_index (display, wnd) {
            if (display == fishDisplayMode.DISPLAY_1O3R || display == fishDisplayMode.DISPLAY_1O1P3R) {
                playerApi.MsSetParam("fishchannelindex", wnd);
                return 1;
            }
            return 0;
        },
        set_panorama_channel_index (display, wnd) {
            if (display == fishDisplayMode.DISPLAY_1O1P3R) {
                playerApi.MsSetParam("panoramachannelindex", wnd);
                return 1;
            }
            return 0;
        },
        channel_is_playing () {
            for (let i=0; i<this.curWndPlay.length; i++) {
                if (this.curWndPlay[i] == 1)
                    return 1;
            }
            return 0;
        },
        get_video_connect_state () {
            let state = [0,0,0,0,0,0,0,0,0];
            let flag = 0;
            for (let i=0; i<this.playUrlIndex.length; i++) {
                if (this.playUrlIndex[i] != -1 && this.curWndPlay[i] == 1) {
                    let content = `${i}:${this.$t('connectExceed')}`;
                    
                    state[i] = playerApi.MsGetConnectState(i);
                    if (state[i] != 0)		//video is playing
                        flag = 1;
                    let indexUrl = this.playUrlIndex[i];
                    if (this.fishDisplay == fishDisplayMode.DISPLAY_1O ||
                        this.fishDisplay == fishDisplayMode.DISPLAY_180P ||
                        this.fishDisplay == fishDisplayMode.DISPLAY_360P)
                        indexUrl = 0;
                    if (state[i] == 1) {
                        if (this.channelPlay[indexUrl] == 0) {
                            this.channelPlay[indexUrl] = 1;
                        }                           
                    } else {
                        if (this.curConn >= this.maxConn && this.maxConn != 0) {
                            this.channelPlay[indexUrl] = 0;
                            playerApi.MsSetParam("limitchannel", content);
                        }
                    }
                }
            }
            if (this.channel_is_playing() == 1) {
                if (flag == 0) {
                    this.playAudio = 0;
                    playerApi.MsSetParam("PlayAudio", this.playAudio==0 ? "0" : "1");
                    this.limitPlayFailed = 1;
                } else if (flag == 1 && this.limitPlayFailed == 1) {
                    this.limitPlayFailed = 0;
                    this.click_audio();
                }
            }
            if (this.getStateFlag) {
                this.getStateTimer = setTimeout( () => {
                    this.get_video_connect_state();
                }, 5000);
            }
        },
        fish_reinit_vca_overlays () {
            for (let i=0; i<this.FISH_MAX_VIEW_NUM; i++) {
                playerApi.MsSetParam("vcawindow",i);
                this.hide_all_lines();
                playerApi.MsSetPersonRectRegion(1, "0:0:0:0:0:0");
                playerApi.MsSetPersonRectWithTime(1, "");
                playerApi.MsSetPolygonMap(0, 1, '-1:-1:-1:-1:-1:-1:-1:-1:-1:-1', '-1:-1:-1:-1:-1:-1:-1:-1:-1:-1');
            }
        },
        get_channel1_window () {
            for (let i=0; i<this.curWndPlay.length; i++) {
                if (this.curWndPlay[i] == 1) {
                    if (this.fishDisplay == fishDisplayMode.DISPLAY_1O) {
                        if (this.playUrlIndex[i] == 0 || this.playUrlIndex[i] == 1)
                            return i;
                    } else {
                        if (this.playUrlIndex[i] == 0)
                            return i;
                    }
                }
            }
            return -1;
        },
        fish_show_vca_overlays () {
            if (!this.isFisheye)
                return ;
            let support = this.fish_support_vca(this.fishCorrect, this.fishDisplay, this.fishInstall);
            if (support) {
                if (this.dewarpType == 1) {
                    this.showOverlay = false;
                } else {
                    this.showOverlay = true;
                }
                this.fish_reinit_vca_overlays();
                let vcaWnd = this.get_channel1_window();
                if (vcaWnd != -1) {
                    playerApi.MsSetParam("vcawindow", vcaWnd);                    
                    let vcaType = ASTGUI.cookies.getCookie("vcaType");
                    if (this.overlaySel != vcaType) {
                        this.overlaySel = IsNull(vcaType)? 0 : parseInt(vcaType);
                    }
                    this.change_overlay(this.overlaySel);
                }
            } else {
                this.drawTimerFlag = false;
                clearTimeout(this.drawTimer);
                this.drawDataTimerFlag = false;
                clearTimeout(this.drawDataTimer);
                this.drawTimerFlag = false;
                clearTimeout(this.drawTimer);
                this.showOverlay = false;
                this.fish_reinit_vca_overlays();
            }
        },
        set_channel_play_state (indexUrl) {
            console.log('set_channel_play_state', indexUrl);
            //disabled substream can not play
            if (this.fishCorrect == fishCorrectMode.MUL_STREAM && (this.fishDisplay == fishDisplayMode.DISPLAY_1O ||
                this.fishDisplay == fishDisplayMode.DISPLAY_180P || this.fishDisplay == fishDisplayMode.DISPLAY_360P))
                if (indexUrl == 1 && this.disableSubChannel)
                    return ;

            this.curUrlListIndex = indexUrl;
            this.click_play();
            playerApi.MsSetParam(`smartstreamon${this.curWndChannel}`, this.smartStreamArr[indexUrl]);
            this.curUrlListIndex = -1;
            if (this.switchFlag == 0) {
                this.getStateFlag = false;
                clearTimeout(this.getStateTimer);
                this.getStateTimer = setTimeout( () => {
                    this.getStateFlag = true;
                    this.get_video_connect_state();
                }, 5000);
            }

            if (this.fishCorrect == fishCorrectMode.MUL_STREAM && (this.fishDisplay == fishDisplayMode.DISPLAY_1O ||
                this.fishDisplay == fishDisplayMode.DISPLAY_360P || this.fishDisplay == fishDisplayMode.DISPLAY_180P)) {
                if (this.scaleSel == 'AUTO')
                    this.change_scale(0);
                else
                    this.change_scale(1);
            }
            playerApi.MsSetParam("livevideoui","1");
            this.fish_show_vca_overlays(indexUrl);
        },
        set_special_channel_play_state (index) {
            if (this.fishSubEnable[this.fishDisplay] == '0') {
                return;
            }

            this.set_channel_play_state(index);
            if (this.zoomEnable) {
                this.zoomEnable = 0;
                playerApi.MsSetParam("zoomenable", "0");
                this.curWndZoom[this.curWndChannel] = this.zoomEnable;
            }

            if (this.playTalk == 1) {
                this.playTalk = 0;
                playerApi.MsStopTalking();
            }
        },
        click_channel (index) {
            if (this.channelPlay.length == 1)
                return;
            this.set_special_channel_play_state(index);
        },
        play_video_by_win_index (win, stream) {
            console.log('play_video_by_win_index', win, stream);
            if (stream == win) {
                if (this.fishDisplay == fishDisplayMode.DISPLAY_1O) {
                    for (let i=fishStreamType.FSH_1O_MAIN; i<=fishStreamType.FSH_1O_SUB; i++) {
                        if (this.fishView[i] != -1) {
                            this.update_select_wnd_status(this.fishView[i]);
                            this.set_channel_play_state(i - fishStreamType.FSH_1O_MAIN);
                            break;
                        }
                    }
                } else if (this.fishDisplay == fishDisplayMode.DISPLAY_360P) {
                    for (let i=fishStreamType.FSH_1P_MAIN; i<=fishStreamType.FSH_1P_SUB; i++) {
                        if (this.fishView[i] != -1) {
                            this.update_select_wnd_status(this.fishView[i]);
                            this.set_channel_play_state(i - fishStreamType.FSH_1P_MAIN);
                            break;
                        }
                    }
                } else if (this.fishDisplay == fishDisplayMode.DISPLAY_180P) {
                    for (let i=fishStreamType.FSH_2P_MAIN; i<=fishStreamType.FSH_2P_SUB; i++) {
                        if (this.fishView[i] != -1) {
                            this.update_select_wnd_status(this.fishView[i]);
                            this.set_channel_play_state(i - fishStreamType.FSH_2P_MAIN);
                            break;
                        }
                    }
                } else if (this.fishDisplay == fishDisplayMode.DISPLAY_4R) {
                    if (this.fishCorrect == fishCorrectMode.MUL_STREAM) {
                        for (let j=0; j<win; j++) {
                            for (let i=fishStreamType.FSH_4R_MAIN; i<=fishStreamType.FSH_4R_FOUR; i++) {
                                if (this.fishView[i] == j) {
                                    this.update_select_wnd_status(this.fishView[i]);
                                    this.set_channel_play_state(i - fishStreamType.FSH_4R_MAIN);
                                    break;
                                }
                            }
                        }
                    } else {
                        this.update_select_wnd_status(0);
                        this.set_channel_play_state(0);
                    }
                } else if (this.fishDisplay == fishDisplayMode.DISPLAY_1O3R) {
                    if (this.fishCorrect == fishCorrectMode.MUL_STREAM) {
                        for (let j=0; j<win; j++) {
                            for (let i=fishStreamType.FSH_1O3R_MAIN; i<=fishStreamType.FSH_1O3R_FOUR; i++) {
                                if (this.fishView[i] == j) {
                                    this.update_select_wnd_status(this.fishView[i]);
                                    this.set_channel_play_state(i - fishStreamType.FSH_1O3R_MAIN);
                                    break;
                                }
                            }
                        }
                    } else {
                        this.update_select_wnd_status(0);
                        this.set_channel_play_state(0);
                    }
                } else if (this.fishDisplay == fishDisplayMode.DISPLAY_1P3R) {
                    for (let j=0; j<win; j++) {
                        for (let i=fishStreamType.FSH_1P3R_MAIN; i<=fishStreamType.FSH_1P3R_FOUR; i++) {
                            if (this.fishView[i] == j) {
                                this.update_select_wnd_status(this.fishView[i]);
                                this.set_channel_play_state(i - fishStreamType.FSH_1P3R_MAIN);
                                break;
                            }
                        }
                    }
                } else if (this.fishDisplay == fishDisplayMode.DISPLAY_1O1P3R) {
                    for (let j=0; j<win; j++) {
                        for (let i=fishStreamType.FSH_1O1P3R_MAIN; i<=fishStreamType.FSH_1O1P3R_FIF; i++) {
                            if (this.fishView[i] == j) {
                                this.update_select_wnd_status(this.fishView[i]);
                                this.set_channel_play_state(i - fishStreamType.FSH_1O1P3R_MAIN);
                                break;
                            }
                        }
                    }
                }
                this.updateFishView = 1;
            } else {
                if (win == 1) {
                    if (this.fishDisplay == fishDisplayMode.DISPLAY_4R) {
                        for (let i=fishStreamType.FSH_4R_MAIN; i<=fishStreamType.FSH_4R_FOUR; i++) {
                            if (this.fishView[i] != -1) {
                                this.update_select_wnd_status(0);
                                this.set_channel_play_state(i - fishStreamType.FSH_4R_MAIN);
                                break;
                            }
                        }
                    } else if (this.fishDisplay == fishDisplayMode.DISPLAY_1O3R) {
                        for (let i=fishStreamType.FSH_1O3R_MAIN; i<=fishStreamType.FSH_1O3R_FOUR; i++) {
                            if (this.fishView[i] != -1) {
                                this.update_select_wnd_status(0);
                                this.set_channel_play_state(i - fishStreamType.FSH_1O3R_MAIN);
                                break;
                            }
                        }
                    } else if (this.fishDisplay == fishDisplayMode.DISPLAY_1P3R) {
                        for (let i=fishStreamType.FSH_1P3R_MAIN; i<=fishStreamType.FSH_1P3R_FOUR; i++) {
                            if (this.fishView[i] != -1) {
                                this.update_select_wnd_status(0);
                                this.set_channel_play_state(i - fishStreamType.FSH_1P3R_MAIN);
                                break;
                            }
                        }
                    } else if (this.fishDisplay == fishDisplayMode.DISPLAY_1O1P3R) {
                        for (let i=fishStreamType.FSH_1O1P3R_MAIN; i<=fishStreamType.FSH_1O1P3R_FIF; i++) {
                            if (this.fishView[i] != -1) {
                                this.update_select_wnd_status(0);
                                this.set_channel_play_state(i - fishStreamType.FSH_1O1P3R_MAIN);
                                break;
                            }
                        }
                    }
                    this.updateFishView = 0;
                } else if (win == 4) {
                    if (this.fishDisplay == fishDisplayMode.DISPLAY_1O) {
                        for (let i=fishStreamType.FSH_1O_MAIN; i<fishStreamType.FSH_1O_SUB; i++) {
                            if (this.fishView[i] != -1) {
                                this.update_select_wnd_status(0);
                                this.set_channel_play_state(i - fishStreamType.FSH_1O_MAIN);
                                break;
                            }
                        }
                    } else if (this.fishDisplay == fishDisplayMode.DISPLAY_360P) {
                        for (let i=fishStreamType.FSH_1P_MAIN; i<fishStreamType.FSH_1P_SUB; i++) {
                            if (this.fishView[i] != -1) {
                                this.update_select_wnd_status(0);
                                this.set_channel_play_state(i - fishStreamType.FSH_1P_MAIN);
                                break;
                            }
                        }
                    } else if (this.fishDisplay == fishDisplayMode.DISPLAY_180P) {
                        for (let i=fishStreamType.FSH_2P_MAIN; i<fishStreamType.FSH_2P_SUB; i++) {
                            if (this.fishView[i] != -1) {
                                this.update_select_wnd_status(0);
                                this.set_channel_play_state(i - fishStreamType.FSH_2P_MAIN);
                                break;
                            }
                        }
                    } else if (this.fishDisplay == fishDisplayMode.DISPLAY_1O1P3R) {
                        for (let j=0; j<win; j++) {
                            for (let i=fishStreamType.FSH_1O1P3R_MAIN; i<fishStreamType.FSH_1O1P3R_FIF; i++) {
                                if (this.fishView[i] == j && this.fishView[i] != 4) {    //remove fifth window
                                    this.update_select_wnd_status(this.fishView[i]);
                                    this.set_channel_play_state(i - fishStreamType.FSH_1O1P3R_MAIN);
                                    break;
                                }
                            }
                        }
                    }
                    this.updateFishView = 0;
                } else if (win == 5) {
                    if (this.fishDisplay == fishDisplayMode.DISPLAY_1O) {
                        for (let i=fishStreamType.FSH_1O_MAIN; i<fishStreamType.FSH_1O_SUB; i++) {
                            if (this.fishView[i] != -1) {
                                this.update_select_wnd_status(0);
                                this.set_channel_play_state(i - fishStreamType.FSH_1O_MAIN);
                                break;
                            }
                        }
                    } else if (this.fishDisplay == fishDisplayMode.DISPLAY_360P) {
                        for (let i=fishStreamType.FSH_1P_MAIN; i<fishStreamType.FSH_1P_SUB; i++) {
                            if (this.fishView[i] != -1) {
                                this.update_select_wnd_status(0);
                                this.set_channel_play_state(i - fishStreamType.FSH_1P_MAIN);
                                break;
                            }
                        }
                    } else if (this.fishDisplay == fishDisplayMode.DISPLAY_180P) {
                        for (let i=fishStreamType.FSH_2P_MAIN; i<fishStreamType.FSH_2P_SUB; i++) {
                            if (this.fishView[i] != -1) {
                                this.update_select_wnd_status(0);
                                this.set_channel_play_state(i - fishStreamType.FSH_2P_MAIN);
                                break;
                            }
                        }
                    } else if (this.fishDisplay == fishDisplayMode.DISPLAY_4R) {
                        for (let j=0; j<win; j++) {
                            for (let i=fishStreamType.FSH_4R_MAIN; i<=fishStreamType.FSH_4R_FOUR; i++) {
                                if (this.fishView[i] == j) {
                                    this.update_select_wnd_status(this.fishView[i] + 1);
                                    this.set_channel_play_state(i - fishStreamType.FSH_4R_MAIN);
                                    break;
                                }
                            }
                        }
                    } else if (this.fishDisplay == fishDisplayMode.DISPLAY_1O3R) {
                        for (let j=0; j<win; j++) {
                            for (let i=fishStreamType.FSH_1O3R_MAIN; i<=fishStreamType.FSH_1O3R_FOUR; i++) {
                                if (this.fishView[i] == j) {
                                    this.update_select_wnd_status(this.fishView[i] + 1);
                                    this.set_channel_play_state(i - fishStreamType.FSH_1O3R_MAIN);
                                    break;
                                }
                            }
                        }
                    } else if (this.fishDisplay == fishDisplayMode.DISPLAY_1P3R) {
                        for (let j=0; j<win; j++) {
                            for (let i=fishStreamType.FSH_1P3R_MAIN; i<=fishStreamType.FSH_1P3R_FOUR; i++) {
                                if (this.fishView[i] == j) {
                                    this.update_select_wnd_status(this.fishView[i] + 1);
                                    this.set_channel_play_state(i - fishStreamType.FSH_1P3R_MAIN);
                                    break;
                                }
                            }
                        }
                    }
                    this.updateFishView = 0;
                }
            }
        },
        window_switch (type) {
            if (type == -1)
                type = this.get_window_type();
            this.showFishDisplaySel = false;
            let windowNum = 0;
            if (type == 0) {
                windowNum = 1;
            } else if (type == 1) {
                windowNum = 4;
            } else if (type == 2) {
                windowNum = 5;
            }

            for (let i=0; i<this.channelPlay.length; i++) {
                this.channelPlay[i] = 0;
            }
            if (xPlayer) {
                xPlayer.initLi(windowNum);
                if (this.fishCorrect == fishCorrectMode.MUL_STREAM && this.fishDisplay > fishDisplayMode.DISPLAY_180P) {
                    tipsBox(this.$t('fishSupport'));
                    return ;
                }
            }
            this.windowsModel = type;
            this.lastWindowsModel = type;
            this.update_fish_window_icon(type);
            this.set_wnd_style(type);
            this.updatePreset = 0;
            this.updateFishView = 0;
            if (this.fishCorrect == fishCorrectMode.MUL_STREAM)
                this.play_video_by_win_index(windowNum, this.channelPlay.length);
            else {
                this.nowStream = -1;
                this.change_stream(this.streamSel);
            }

            this.updatePreset = 1;
            this.update_select_wnd_status(0);

            if (this.switchFlag == 1 || this.switchFlag == 2) {
                this.getStateFlag = false;
                clearTimeout(this.getStateTimer);
                this.getStateTimer = setTimeout( () => {
                    this.getStateFlag = true;
                    this.get_video_connect_state();
                }, 15000);
            } else if (this.switchFlag == 3) {
                this.getStateFlag = false;
                clearTimeout(this.getStateTimer);
                this.getStateTimer = setTimeout( () => {
                    this.getStateFlag = true;
                    this.get_video_connect_state();
                }, 10000);
            }
            this.switchFlag = 0;
        },
        show_switch_loading_view_by_state () {
            let state = 0, flag = 0;
            for (let i=0; i<this.channelPlay.length; i++) {
                if (this.playUrlIndex[i] != -1 && this.curWndPlay[i] == 1) {
                    state = playerApi.MsGetConnectState(i);
                    if (state != 1) {
                        flag = 1;
                        break;
                    }
                }
            }
            if (flag == 0) {
                this.setLoadingTimes = 0;
                this.loading.close();
                this.isLoadingView = 0;
            } else {
                if (this.setLoadingTimes >= 9) {
                    this.setLoadingTimes = 0;
                    this.isLoadingView = 0;
                    this.loading.close();
                    return ;
                }
                this.switchLoadingTimer = setTimeout( () => {
                    this.show_switch_loading_view_by_state();
                }, 1000);
                this.setLoadingTimes++;
            }
        },
        show_switch_loading_view () {
            this.switchLoadingTimer = setTimeout( () => {
                this.show_switch_loading_view_by_state();
            }, 1000);
            this.isLoadingView = 1;
            this.loading = show_loading();
        },
        update_sub_and_smart_stream () {
            let url = 'paratest=enablesubstream&paratest=smartstreamswitch&paratest=subsmartstream' +
                '&paratest=thirdsmartstream&paratest=fourthsmartstream&paratest=fifthsmartstream';
            this.subSmartTimer = setTimeout( () => {
                send_vb_htm(url).then( res => {
                    let txt = res;
                    let ix = txt.indexOf('OK enablesubstream=');
                    if (ix >= 0) {
                        let code = txt.substring(ix + 19);
                        this.subStreamEnable = parseInt(code);
                        if (this.fishCorrect == fishCorrectMode.JOINT_VIDEO) {
                            this.init_stream_options();
                            this.streamSel = this.nowStream;
                            if (this.subStreamEnable == 0) {
                                if (this.nowStream == 1) {
                                    this.nowStream = -1;
                                    this.streamSel = 0;
                                    this.change_stream(0);
                                }
                            }
                        } else {
                            if (this.subStreamEnable == 1) {
                                if (this.channelShowSub) {
                                    this.disableSubChannel = false;
                                }
                            } else {
                                if (this.channelShowSub)
                                    this.disableSubChannel = true;
                                this.streamSel = 0;
                            }
                        }
                    }
                    ix = txt.indexOf('OK smartstreamswitch=');
                    if (ix >= 0) {
                        let code = txt.substring(ix + 21);
                        this.smartStreamArr[0] = parseInt(code);
                    }
                    ix = txt.indexOf('OK subsmartstream=');
                    if (ix >= 0) {
                        let code = txt.substring(ix + 18);
                        this.smartStreamArr[1] = parseInt(code);
                    }
                    ix = txt.indexOf('OK thirdsmartstream=');
                    if (ix >= 0) {
                        let code = txt.substring(ix + 20);
                        this.smartStreamArr[2] = parseInt(code);
                    }
                    ix = txt.indexOf('OK fourthsmartstream=');
                    if (ix >= 0) {
                        let code = txt.substring(ix + 21);
                        this.smartStreamArr[3] = parseInt(code);
                    }
                    ix = txt.indexOf('OK fifthsmartstream=');
                    if (ix >= 0) {
                        let code = txt.substring(ix + 20);
                        this.smartStreamArr[4] = parseInt(code);
                    }
                    for (let i=0; i<this.FISH_MAX_VIEW_NUM; i++) {
                        if (this.curWndPlay[i] == 1)
                            playerApi.MsSetParam(`smartstreamon${i}`, this.smartStreamArr[this.playUrlIndex[i]]);
                    }
                });
            }, 2000);
        },
        disable_smart_event (enable) {
            if (enable & vcaEnable.INTRUSION_ENTER_ENABLE) {
                let url = `&intrusionenterenable4=0&intrusionenterenable1=0` +
                    `&intrusionenterenable2=0&intrusionenterenable3=0`;
                set_page_data('msvca', url);
            }
            if (enable & vcaEnable.INTRUSION_EXIT_ENABLE) {
                let url = `&intrusionexitenable4=0&intrusionexitenable1=0` + 
                    `&intrusionexitenable2=0&intrusionexitenable3=0`;
                set_page_data('msvca', url);
            }
            if (enable & vcaEnable.ADVANCED_MOTION_ENABLE) {
                let url = `&advancedmotionenable4=0&advancedmotionenable1=0` +
                    `&advancedmotionenable2=0&advancedmotionenable3=0`;
                set_page_data('msvca', url);
            }
            if (enable & vcaEnable.TAMPER_DETECTION_ENABLE) {
                let url = '&ctdenable=0';
                set_page_data('msvca', url);
            }
            if (enable & vcaEnable.LINE_CROSSING_ENABLE) {
                let url = '&lineenable1=0&lineenable2=0&lineenable3=0&lineenable4=0';
                set_page_data('msvca', url);
            }
            if (enable & vcaEnable.LOITERING_ENABLE) {
                let url = `&loiteringenable1=0&loiteringenable2=0` +
                    `&loiteringenable3=0&loiteringenable4=0`;
                set_page_data('msvca', url);
            }
            if (enable & vcaEnable.HUMMAN_DETECTION_ENABLE) {
                let url = '&human=0';
                set_page_data('msvca', url);
            }
            if (enable & vcaEnable.COUNTER_ENABLE) {
                let url = '&counterenable=0';
                set_page_data('vcacount', url);
            }
            if (enable & vcaEnable.LEFT_ENABLE || enable & vcaEnable.REMOVE_ENABLE) {
                let url = `&objectleftenable1=0&objectremoveenable1=0` + 
                    `&objectleftenable2=0&objectremoveenable2=0` +
                    `&objectleftenable3=0&objectremoveenable3=0` +
                    `&objectleftenable4=0&objectremoveenable4=0`;
                set_page_data('msvca', url);
            }
            if (enable & vcaEnable.REGIONAL_PEOPLE_COUNTING_ENABLE) {
                let url = '&regionalEnable=0;0;0;0';
                set_page_data('regionalpeople', url);
            }
        },
        auto_track_disable_other_event (smartEventEnable, ptzMotionEnable, lprEnable, ptzMotionEnableVca) {
            let url;
            if (lprEnable != 0) {
                let url = '&lprenable=0';
                set_page_data('mslpr', url);
            }
            if (ptzMotionEnable != 0) {
                url = '&ptzmotionenable=0';
                set_page_data(`alarm.${alarmEvent.ALARM_EVENT_MOTION}`, url);
            }
            if (ptzMotionEnableVca != 0) {
                url = '&ptzmotionenable=0';
                set_page_data(`alarm.${alarmEvent.ALARM_EVENT_VCA_ADVANCED_MOTION}`, url);
            }
            if (this.sysInfo.supportcroproi == '1' || this.isFisheye) {
                this.disable_smart_event(smartEventEnable);
            }
        },
        set_fish_track_state () {
            if (this.fish_support_show_track(this.fishInstall, this.fishDisplay) == 1 && is_support_ptz_control()) {
                this.disableTrack = false;
                if (this.trackEnable == 1 && this.showTrack == 1) {
                    if (this.smartEventEnable &&
                        this.fish_support_vca(this.fishCorrect, this.fishDisplay, this.fishInstall) == 1) {
                        this.auto_track_disable_other_event(this.smartEventEnable, this.ptzMotionEnable,
                            this.lprEnable);
                        this.smartEventEnable = 0;
                        this.ptzMotionEnable = 0;
                        this.lprEnable = 0;
                    }
                }
            } else {
                this.disableTrack = true;
            }
            this.show_auto_track();
        },
        set_soft_dewarp_install (install) {
            this.curFishInstall = install;
            playerApi.MsSetParam("fisheyeSoftwareInstallModel", install);
            this.build_display_arr_by_install(install);
            playerApi.MsSetParam("fisheyeDisplayModel", "0");
            let display = parseInt(playerApi.MsGetParam("fisheyeDisplayModel"));
            this.display_switch(display);
        },
        send_fish_install (type) {
            if (!this.isFisheye)
                return ;
            let correct = playerApi.MsGetParam("fisheyeCorrectionModel");
            if (correct == this.G_SOFT_DEWARP) {
                this.set_soft_dewarp_install(type);
                let install = ASTGUI.cookies.getCookie("fisheyeSoftwareInstallModel");
                if (install != type) {
                    ASTGUI.cookies.setCookie("fisheyeSoftwareDisplayModel", "0");
                }
                ASTGUI.cookies.setCookie("fisheyeSoftwareInstallModel", String(type));
                return ;
            }
            playerApi.MsSetParam("fishinstallmodel", type);
            let time = new Date();
            let curTime = parseInt(time.getTime());
            if (curTime - this.lastSwitchTime >= this.switchInterval * 1000 || this.isLoadingView == 0)
                this.lastSwitchTime = curTime;
            else
                return ;
            if (this.fishInstall == type)
                return ;
            this.close_all_stream_display();
            this.curConn -= this.get_playing_channel_num();
            if (this.curConn < 0) {
                this.curConn = 0;
            }
            this.isPlaying = 0;
            let url = `&fishinstallmodel=${type}`;

            this.fish_data_init();
            this.curFishInstall = this.fishInstall = type;
            this.build_display_arr_by_install(type);
            if (this.fishCorrect == fishCorrectMode.JOINT_VIDEO &&
                playerApi.MsGetParam("fisheyeCorrectionModel") != '1')
                this.showDisplay1O3R = false;
            //click dispaly switch,change model to fisheye
            if (this.fishInstall == fishInstallMode.WALL && this.fishDisplay == fishDisplayMode.DISPLAY_180P) {
                this.fishDisplay = 0;
                this.curDisplay = 0;
            }
            url += `&fishdisplaymodel=${this.fishDisplay}`;
            {
                this.close_all_stream_display();
                playerApi.MsSetParam("hasclick", "0");	//for winplugin
                this.set_correct_model(this.fishDisplay);
                playerApi.MsSetParam("fishinstallmodel", this.fishInstall);
                playerApi.MsSetParam("fishdisplayplaymodel", this.fishDisplay);
                this.display_switch(this.fishDisplay);
                this.build_channel_list(this.get_url_count(this.fishDisplay));
                this.stopAllFishVideo = 0;
                this.switchFlag = 1;
                this.change_scale(0);
                setTimeout( () => {
                    this.window_switch(-1);
                }, 200);
                if (!this.bPtzTimer)
                    this.get_fish_ptz_region();
            }
            
            set_page_data('fish', url);
            if (this.isFisheye) {
                this.$store.dispatch("fisheye/setFishInstall", this.fishInstall);
                this.$store.dispatch("fisheye/setFishDisplay", this.fishDisplay);
            }
            this.autoScanStatus = 0;
            for (let i=0; i<this.FISH_MAX_VIEW_NUM; i++)
                this.fishAutoScan[i] = 0;
            this.show_switch_loading_view();
            this.getFishTime = 0;
            this.update_sub_and_smart_stream();
            this.close_talk();
            this.set_fish_track_state();
        },
        send_fish_display (type) {
            if (!this.isFisheye)
                return ;
            let correct = playerApi.MsGetParam("fisheyeCorrectionModel");
            if (correct == '1') {
                ASTGUI.cookies.setCookie("fisheyeSoftwareDisplayModel", String(type)); 
                playerApi.MsSetParam("fisheyeDisplayModel", String(type));
                this.display_switch(type);
                return ;
            }
            let time = new Date();
            let curTime = parseInt(time.getTime());
            if (curTime - this.lastSwitchTime >= this.switchInterval * 1000 || this.isLoadingView == 0)
                this.lastSwitchTime = curTime;
            else
                return ;
            if (this.curDisplay == type)
                return ;
            
            if (this.fishCorrect == fishCorrectMode.MUL_STREAM && type > fishDisplayMode.DISPLAY_180P &&
                !isNull(xPlayer)) {
                tipsBox(this.$t('fishSupport'));
                return ;
            }
            this.curDisplay = type;
            this.close_all_stream_display();
            this.curConn -= this.get_playing_channel_num();
            if (this.curConn < 0) {
                this.curConn = 0;
            }
            this.isPlaying = 0;
            let url = `&fishdisplaymodel=${type}`;
            set_page_data('fish', url);
            this.$store.dispatch("fisheye/setFishDisplay", type);

            playerApi.MsSetParam("hasclick", "0");
            this.set_correct_model(type);
            playerApi.MsSetParam("fishdisplayplaymodel", type);

            this.fish_data_init();
            this.fishDisplay = type;
            this.display_switch(this.fishDisplay);
            this.build_channel_list(this.get_url_count(this.fishDisplay));
            this.stopAllFishVideo = 0;
            this.switchFlag = 2;
            this.change_scale(0);
            if (this.fishCorrect == fishCorrectMode.JOINT_VIDEO && this.fishSubEnable[type] == '0' && this.nowStream == 1) {
                this.subStreamEnable = 0;
                this.init_stream_options();
                this.nowStream = -1;
                this.streamSel = 0;
                this.change_stream(this.streamSel);
            }
            setTimeout( () => {
                this.window_switch(-1);
            }, 200);
            if (!this.bPtzTimer)
                this.get_fish_ptz_region();

            for (let i=0; i<this.FISH_MAX_VIEW_NUM; i++) {
                this.fishAutoScan[i] = 0;
            }
            for (let i=0; i<this.curWndZoom.length; i++) {
                this.curWndZoom[i] = 0;
                this.recordStartFish[i] = 0;
            }
            this.zoomEnable = 0;
            this.isRecording = 0;
            this.set_channel_record(0);
            this.show_switch_loading_view();
            this.getFishTime = 0;
            this.update_sub_and_smart_stream();
            this.close_talk();
            this.set_fish_track_state();
        },
        update_fish_window_icon (type) {
            switch (type) {
                case 0:
                    this.windowIcon = 'icon-play2';
                    break;
                case 1:
                    this.windowIcon = 'icon-display5';
                    break;
                case 2:
                    this.windowIcon = 'icon-fives';
                    break;
            }
        },
        send_fish_windows (type) {
            if (!this.isFisheye)
                return ;
            if (this.fishCorrect == fishCorrectMode.JOINT_VIDEO)
                return ;
            if (this.lastWindowsModel == type)
                return ;
            this.close_all_stream_display();
            this.curConn -= this.get_playing_channel_num(); //use for judging connection limit
            if (this.curConn < 0) {
                this.curConn = 0;
            }
            this.isPlaying = 0;
            this.fish_data_init();
            this.stopAllFishVideo = 0;  //not reset
            this.switchFlag = 3;
            this.change_scale(0);
            this.window_switch(type);
            playerApi.MsSetParam("hasclick", "0");	//for winplugin
            this.close_talk();
        },
        is_fish_software_correct() {
            if (ASTGUI.cookies.getCookie('fisheyeCorrectionModel') == '1' ||
                playerApi.MsGetParam("fisheyeCorrectionModel") == '1')
                return true;
            else
                return false;
        },
        build_display_arr_by_install (install) {
            this.displayArr = [0, 1];
            if (install != fishInstallMode.WALL)
                this.displayArr.push(2);
            this.displayArr.push(4);
            this.displayArr.push(5);
            if (this.dewarpType == 0) {
                this.displayArr.push(6);
                if (this.fishCorrect == fishCorrectMode.MUL_STREAM)
                    this.displayArr.push(7);
            } else {
                this.displayArr.push(8);
                this.displayArr.push(9);
                if (install != fishInstallMode.WALL) {
                    this.displayArr.push(10);
                    this.displayArr.push(11);
                }
            }
        },
        correction_model_switch (val) {
            if (!this.isFisheye)
                return ;
            let display = this.fishDisplay, install = this.fishInstall, type = -1;
            if (val == 1) {
                if (display != fishDisplayMode.DISPLAY_1O && display != fishDisplayMode.DISPLAY_1O3R &&
                    display != fishDisplayMode.DISPLAY_1O1P3R) {
                    if (this.is_fish_software_correct()) {
                        playerApi.MsSetParam("fisheyeCorrectionModel", "0");
                        ASTGUI.cookies.setCookie("fisheyeCorrectionModel", "0");
                        this.correction_model_switch(0);
                    }
                    confirmBox(this.$t('changeToO'), () => {
                        this.send_fish_display(0);
                        if (this.fishDisplay != fishDisplayMode.DISPLAY_1O)
                            return ;
                        this.correction_model_switch(1);
                    });
                    return ;
                }
                this.update_select_wnd_status(1);
                this.showOverlay = false;
                playerApi.MsSetParam("fisheyeCorrectionModel", val);
                this.dewarpType = 1;
                install = ASTGUI.cookies.getCookie("fisheyeSoftwareInstallModel");
                if (install == '' || install == null || install == undefined)
                    install = this.fishInstall;
                playerApi.MsSetParam("fisheyeSoftwareInstallModel", install);

                display = parseInt(ASTGUI.cookies.getCookie("fisheyeSoftwareDisplayModel"));
                if (isNaN(display)) {
                    display = 0;
                    ASTGUI.cookies.setCookie("fisheyeSoftwareDisplayModel", display);
                }
                playerApi.MsSetParam("fisheyeDisplayModel", String(display));
                type = 0;
                ASTGUI.cookies.setCookie("fisheyeCorrectionModel", "1");
                if (this.fishCorrect == fishCorrectMode.MUL_STREAM)
                    this.showFishWindow = false;
            } else if (val == 0) {
                playerApi.MsSetParam("fisheyeCorrectionModel", val);
                this.dewarpType = 0;
                type = -1;
                ASTGUI.cookies.setCookie("fisheyeCorrectionModel", "0");
                if (this.fishCorrect == fishCorrectMode.MUL_STREAM)
                    this.showFishWindow = true;
                this.showOverlay = true;
            }
            this.curFishInstall = install;
            this.build_display_arr_by_install(install);

            this.curDisplay = display;
            if (playerApi.MsGetParam("fisheyeCorrectionModel") != '0')
                this.showPlus = false;
            else
                this.showPlus = true;
            if (this.fishCorrect == fishCorrectMode.JOINT_VIDEO &&
                playerApi.MsGetParam("fisheyeCorrectionModel") != '1')
                this.showDisplay1O3R = false;
            if (this.fishCorrect == fishCorrectMode.MUL_STREAM) {
                if (type == -1) {
                    type = this.windowsModel;
                    this.windowsModel = type;
                    this.lastWindowsModel = type;
                }
                this.update_fish_window_icon(type);
                playerApi.MsSetParam("fisheyeWndStyle", this.get_win_by_channel(type)); 
            }
            if (this.isRecording == 1)
                this.click_record();
            this.set_fish_track_state();
        },
        init_fish_view () {
            let pos = ASTGUI.cookies.getCookie("fishposition");
            if (IsNull(pos)) {
                pos = '0;0;0;0;0;0;0;1;2;3;0;1;2;3;0;1;2;3;1;0;2;3;4;';
                if (Judge_Browser_Ver() === 0)
                    pos = '-1;0;-1;0;-1;0;0;1;2;3;0;1;2;3;0;1;2;3;1;0;2;3;4;';
                ASTGUI.cookies.setLoginCookie("fishposition", pos);	//reserve 1000 days
            }
            this.fishView = pos.split(';');
            for (let i=0; i<fishStreamType.FSH_MAX; i++) {
                this.fishView[i] = parseInt(this.fishView[i]);
            }
            if (this.fishCorrect == fishCorrectMode.MUL_STREAM) {
                if (this.subStreamEnable == 0) {
                    if (this.fishDisplay == fishDisplayMode.DISPLAY_1O) {
                        if (this.fishView[fishStreamType.FSH_1O_SUB] == 0) {
                            this.fishView[fishStreamType.FSH_1O_SUB] = -1;
                            this.fishView[fishStreamType.FSH_1O_MAIN] = 0;
                        }
                    } else if (this.fishDisplay == fishDisplayMode.DISPLAY_360P) {
                        if (this.fishView[fishStreamType.FSH_1P_SUB] == 0) {
                            this.fishView[fishStreamType.FSH_1P_SUB] = -1;
                            this.fishView[fishStreamType.FSH_1P_MAIN] = 0;
                        }
                    } else if (this.fishDisplay == fishDisplayMode.DISPLAY_180P) {
                        if (this.fishView[fishStreamType.FSH_2P_SUB] == 0) {
                            this.fishView[fishStreamType.FSH_2P_SUB] = -1;
                            this.fishView[fishStreamType.FSH_2P_MAIN] = 0;
                        }
                    }
                }
            }
        },
        get_fish_mouse_move (stream, pan, tile, nZoom, operate) {
            let url = `&ipncptz=fishptzposition=${stream}:0:${pan}:${tile}:${nZoom}:`;
            let flag = 0;
            //if zoom change,stop auto sacn digital label(stop get ptz)
            let ptz = this.fishPtzRegion.split(';');
            let zoom = ptz[stream].split(':');
            if (nZoom != zoom[2]) {
                this.update_digital_label(0);
                flag = 1;
            }
            url += `${operate};`;      //0:drug 1:zoom

            let time = new Date();
            let curTime = parseInt(time.getTime());
            if (curTime - this.lastFishMove <= 100 && flag == 0)
                return ;
            else
                this.lastFishMove = curTime;
            send_vb_htm(url);
        },
        var_callback() {
            this.init_stream_options();
            loadOBJ();
            check_over_max_conn(true);
            this.isPlaying = 1;
            if (this.sysInfo.isfisheye == '1') {
                if (this.fishCorrect == fishCorrectMode.MUL_STREAM)
                    this.showStream = false;
                if (xPlayer)
                    this.showDewarpBox = false;
                else {
                    registerIeFun(1, this.get_fish_mouse_move);
                    registerIeFun(2, this.update_select_wnd_status);
                }
                this.nowStream = parseInt(ASTGUI.cookies.getCookie('msStreamIndex'));
                let i;
                for (i=0; i<this.streamOptions.length; i++) {
                    if (this.nowStream == this.streamOptions[i].value)
                        break;
                }
                if (i == this.streamOptions.length)
                    this.nowStream = 0;
                if (Judge_Browser_Ver() === 0) {
                    if (this.nowStream == 0 && this.subStreamEnable)
                        this.nowStream = 1;
                }
                this.streamSel = this.nowStream;

                let correct = ASTGUI.cookies.getCookie("fisheyeCorrectionModel");
                if (correct == '' || correct == null || correct == undefined)
                    correct = '0';
                this.correction_model_switch(correct);

                if (correct == '1')
                    this.showPlus = false;
                else
                    this.showPlus = true;
                playerApi.MsSetParam("fisheyeosdzmtime", this.osdZmTime);
                playerApi.MsSetParam("fisheyeosdmdtime", this.osdMdTime);
                playerApi.MsSetParam("fisheyeosdpatroltime", this.osdPatrolTime);
                playerApi.MsSetParam("fisheyeosdscantime", this.osdScanTime);
                playerApi.MsSetParam("fisheyeCmdSpeed", this.ptzSpeed);
            } else {
                if (!xPlayer && this.showPtzPanel) {
                    registerIeFun(0, Get3dPosButtonUp);
                    registerIeFun(3, GetZoomCmd);
                }
                this.streamSel = parseInt(ASTGUI.cookies.getCookie('msStreamIndex'));
                let i;
                for (i=0; i<this.streamOptions.length; i++) {
                    if (this.streamSel == this.streamOptions[i].value)
                        break;
                }
                if (i == this.streamOptions.length)
                    this.streamSel = 0;
                if (Judge_Browser_Ver() === 0) {
                    if (this.streamSel == 0 && this.subStreamEnable)
                        this.streamSel = 1;
                }
                this.nowStream = this.streamSel;
                console.log('this.streamSel', this.streamSel);
                this.change_stream(this.streamSel);
                this.show_auto_track();
            }

            if (browser_IE) {
                let protocol = playerApi.MsGetParam('protocol');
                if (!IsNull(protocol) && this.is_support_protocol(protocol)) {
                    this.protocolSel = parseInt(protocol);
                } else {
                    this.protocolSel = 0;
                }
                let smooth = playerApi.MsGetParam('smoothstream');
                if (!IsNull(smooth)) {
                    this.smoothSel = parseInt(smooth);
                } else {
                    this.smoothSel = 0;
                }
            }
            if (this.liveVideoPri) {
                PlayActiveXM(this.streamSel, this.get_smart_stream(this.streamSel),
                    this.update_select_wnd_status, this.get_fish_mouse_move, this.set_correct_model, this);
            }
            if (this.isFisheye) {
                this.build_channel_list(this.get_url_count(this.fishDisplay));
                this.switchFlag = 3;
                this.window_switch(-1);
                this.disablePTZFeature = true
            }
            this.load_overlays();
            if ((this.sysInfo.isspeed == '1' || this.sysInfo.isptzbullet == '1' || this.sysInfo.isminiptzdome == '1')
                && !this.isAnony && this.isSupportPtzCtrl)
                this.init_3d_pos();
            else {
                this.posEnable = false
            }
            this.isLoaded = true;
            ////this.player = playerApi.MsGetPlayer();
            //
            let pwmInter = parseInt(this.sysInfo.whiteirpwminterface);
            if(pwmInter === pwmType.PWM_TYPE_1IR_1W || pwmInter === pwmType.PWM_TYPE_2IR_1W
                || pwmInter === pwmType.PWM_TYPE_1IR_2W) {
                this.isShowLight = true;
                this.lightTimer = setTimeout (() => {
                    this.update_light_status();
                }, 1000);
            }
        },
        sys_info_cb (obj) {
            this.sysInfo = obj;
            if (obj.isnewhi3516dv300 == '1' || obj.iscv2xplatform == '1')
                this.sysInfo.isAiPlatform = 1;
            if (obj.supportlpr == '1') {
                this.hasLprPermission = is_support_lpr_permission();
                this.isLpr = true;
            }
            if ((obj.lessalarminterface == '1' && obj.lessalarmextinterface == '1') ||
                is_support_event_permission() == false) {
                this.hasExtout = false;
            }
            if ((obj.lessalarminterface == '1' && obj.lessalarmextinterface == '1') ||
             (obj.isspeed == '1' && obj.isnewspeed == '0')) {
                this.manualoutputSupport = false;
            }
            if (obj.lessalarmextinterface == '0')
                this.showExt = true;
            if (obj.isspeeddm == '1' ||
                (obj.isspeed == '0' && obj.isminiptzdome == '0' && obj.isptzbulletv3 == '0'))
                this.showDefog = false;
            if (obj.supportface == '1') {
                this.initFaceShow();
            }
            if (obj.isnewhi3516dv300 == '0' && obj.iscv2xplatform == '0')
                this.overlayOptions.splice(9, 1);
            if (obj.isusingcropvideo == '0')
                this.overlayOptions.splice(8, 1);
            if (!(obj.isautofocus == '1' || obj.isspeed == '1' || obj.isptzbullet == '1' || obj.isbox == '1' ||
                obj.isautofocusnew == '1' || obj.isabfbox == '1' || obj.isminiptzdome == '1')) {
                this.showZoomPos = false;
                this.isAutoF = false;
            } else {
                if (obj.isautofocusnew == '1') {
                    this.showZoomPos = false;
                    this.isAutoFNew = true;
                    this.irisMode = 1;
                }
            }
            if (obj.isspeed == '1' || obj.isptzbullet == '1' || obj.isfisheye == '1' ||
                obj.isbox == '1' || obj.isminiptzdome == '1') {
                this.showZoomPos = false;
            }

            if (obj.isspeed == '1' || obj.isptzbullet == '1' || obj.isfisheye == '1' || obj.isbox == '1'
                || obj.isminiptzdome == '1') {
                if (!this.isForbidPtzCtrlAndSettings) {
                    this.showPtzPanel = true;
                    this.activeTab = 'ptzTab';
                }
                this.isAutoF = true;
            }
            if (obj.isfisheye == '1') {
                if (!is_viewer() && (is_support_rtsp_access() || is_support_live_view())) {
                    this.activeTab = 'fishTab';
                    this.showFishPanel = true;
                }
                this.isFisheye = true;
            }
            if (obj.supportvca == '1' && obj.isfisheye == '0' && obj)
                this.showVcaInstall = true;
            if (obj.isspeed == '1' || obj.isptzbullet == '1' || obj.isminiptzdome == '1') {
                this.showVcaInstall = false;
            }
            
            if (this.sysInfo.supportlpr == '1') {
                // if (ASTGUI.cookies.getCookie("lprmode") == null || ASTGUI.cookies.getCookie("lprmode") == '1') {
                //     this.isLprMode = true;
                // } else {
                //     this.isLprMode = false;
                // }
                // this.isLprMode = !this.isLprMode;
                // this.change_lpr_mode();
                if (this.$route.path.indexOf('liveview/lpr') != -1)
                    this.change_lpr_mode();
            }

            if (is_viewer()) {
                this.showImagePanel = false
                if (this.isForbidPtzCtrlAndSettings) {
                    this.showPtzPanel = false
                    this.showTriggleRightBtn = false
                    this.showFaceBtn = false
                    this.showRight = false
                }
            } else {
                this.showImagePanel = this.showZoomPos || (!this.showPtzPanel) || this.liveVideoPri
            }
            if (this.isForbidPtzCtrlAndSettings) {
                //this.showZoomPos = false
                this.showPtzPanel = false
                this.disableTrack = true
                this.disablePTZFeature = true
            }

            if (this.sysInfo.isbox == '1') {
                this.disableZoom = true;
                this.disableFocus = true;
                this.disableLens = true;
                this.disableAutoFocus = true;
                this.disableAutoIris = true;
                this.isForbidPtzCtrlAndSettings = true;
                this.disablePTZFeature = true;
                this.disableTrack = true;
                if (this.sysInfo.isabfbox == '1') {
                    this.disableFocus = false;
                    this.disableLens = false;
                    this.disableAutoFocus = false;
                    this.disableAutoIris = false;
                }
                if (this.irisType == 1) {
                    this.noManualIris = true;
                }
            }
            this.init_preset();
        },
        change_ptz_speed () {
            let url = `&ptzspeed=${this.ptzSpeed-1}`;
            send_vb_htm(url);
            playerApi.MsSetParam("fisheyeCmdSpeed", this.ptzSpeed);
        },
        send_iris_in () {
            let url = '&irisstep=2&irismanualstep=1&ipncptz=irisin';
            send_vb_htm(url);
        },
        send_iris_out () {
            let url = '&irisstep=2&irismanualstep=0&ipncptz=irisout';
            send_vb_htm(url);
        },
        send_iris (inOut) {
            if (inOut) {
                this.send_iris_in();
                this.irisTimer = setTimeout( () => {
                    this.send_iris_in();
                }, 400);
            } else {
                this.send_iris_out();
                this.irisTimer = setTimeout( () => {
                    this.send_iris_out();
                }, 400);
            }
        },
        change_iris_mode () {
            if (this.noAutoIris) { // 不支持配置自动光圈
                return
            }
            if (this.caniris == 0 && this.sysInfo.isbox == '0') {
                return;
            }
            this.irisMode = 1 - this.irisMode;
            let url = `&irismode=${this.irisMode}`;
            send_vb_htm(url);
        },
        update_light_status () {
            let url = 'paratest=lightstatus';
            send_vb_htm(url).then( res => {
                let txt = res;
                let str = 'OK lightstatus=';
                let ix = txt.indexOf(str);
                if (ix >= 0) {
                    this.lightMode = parseInt(txt.substr(ix + str.length, 1));
                    if (this.lightMode == 0) {
                        clearTimeout(this.lightTimer);
                        this.lightTimer = null;
                        return ;
                    }
                }
                this.lightTimer = setTimeout (() => {
                    this.update_light_status();
                }, 1000);
            });
        },
        change_light () {
            this.lightMode = 1 - this.lightMode;
            if (this.lightMode) {
                let url = '&ipncptz=lighton';
                send_vb_htm(url);
                clearTimeout(this.lightTimer);
                this.lightTimer = null;
                this.lightTimer = setTimeout( () => {
                    this.update_light_status();
                }, 2000);
            } else {
                let url = '&ipncptz=lightoff';
                clearTimeout(this.lightTimer);
                this.lightTimer = null;
                send_vb_htm(url);
            }
        },
        change_pos () {
            if ((this.sysInfo.isspeed == '1' || this.sysInfo.isptzbullet == '1' ||
                 this.sysInfo.isminiptzdome == '1')
                && this.zoomEnable) {
                tipsBox(this.$t('zoomWithout3d'), 'warning');
                return ;
            }
            this.posEnable = 1 - this.posEnable;
            let url = `&pos3denable=${this.posEnable}`
            if (this.posEnable) {
                url += '&trackmanualenable=0';
                this.manualTrack = 0;
                set_manual_track(0);
            }
            playerApi.MsSetParam('platform', 1);
            playerApi.MsSet3dPosRegion(this.posEnable, '0:0:0:0');
            send_vb_htm(url);
        },
        init_3d_pos () {
            playerApi.MsSetParam("platform", 1);
            if (this.trackEnable && this.manualTrack)
                playerApi.MsSet3dPosRegion(this.manualTrack, "0:0:0:0");
            else
                playerApi.MsSet3dPosRegion(this.posEnable, "0:0:0:0");
        },
        update_patrol_status () {
            let url = 'paratest=onepatrol';
            send_vb_htm(url).then( res => {
                let txt = res;
                let str = 'OK onepatrol=';
                let ix = txt.indexOf(str);
                if (ix >= 0) {
                    let patrolStatus = parseInt(txt.substr(ix + str.length, 1));
                    this.patrolMode = patrolStatus;
                    if (patrolStatus == 0) {
                        if (this.patrolRecover) {
                            this.patrolTimer = setTimeout( () => {
                                this.update_patrol_status();
                            }, 3000);
						} else {
                            clearTimeout(this.patrolTimer);
                            this.patrolTimer = null;
                        }
                        return ;
                    }
                }
                this.patrolTimer = setTimeout( () => {
                    this.update_patrol_status();
                }, 1000);
            });
        },
        change_patrol () {
            if (this.is_rec_or_limit()) {
                return;
            }
            if (this.presetArr.length < 18)
                return ;
            if (this.sysInfo.isptzdevicewithouttour == '1' || this.isfisheye == '1')
                return ;
            this.patrolMode = 1 - this.patrolMode;
            if (this.patrolMode) {
                let url = '&ipncptz=enablepatrol';
                send_vb_htm(url);
                clearTimeout(this.patrolTimer);
                this.patrolTimer = null;
                this.patrolTimer = setTimeout( () => {
                    this.update_patrol_status();
                }, 1000);
            } else {
                send_ptz_cmd_to_device(0);
                clearTimeout(this.patrolTimer);
                this.patrolTimer = null;
            }
        },
        change_watch () {
            if (this.sysInfo.isfisheye == '1') {
                return ;
            }
            this.watchEnable = 1 - this.watchEnable;
            let url = `&ipncptz=savewatch=${this.watchEnable}:${this.watchTime}:${this.watchMode}:`;
            let i=0;
            for (; i<this.presetArr.length; i++) {
                if (this.watchId == this.presetArr[i])
                    break;
            }
            if (i == this.presetArr.length)
                url += 0;
            else
                url += this.watchId;
            send_vb_htm(url);
        },
        get_auto_track_tips () {
            let tip = '';
            if (this.sysInfo.supportcroproi == '1' || this.isFisheye) {
                if (this.smartEventEnable && this.ptzMotionEnable || this.ptzMotionEnableVca)
                    tip = this.$t('smartMotionDisableTips');
                else if (this.smartEventEnable)
                    tip = this.$t('smartDisableTips');
                else if (this.ptzMotionEnable || this.ptzMotionEnableVca)
                    tip = this.$t('motionDisableTips');
                else if (this.lprEnable)
                    tip = this.$t('lprDisableTips');
            } else {
                if (this.lprEnable && (this.ptzMotionEnable || this.ptzMotionEnableVca))
                    tip = this.$t('lprSmartDisableTips');
                else if (this.ptzMotionEnable || this.ptzMotionEnableVca)
                    tip = this.$t('motionDisableTips');
                else if (this.lprEnable)
                    tip = this.$t('lprDisableTips');
                else if ((this.sysInfo.isnewhi3516dv300 == '1' || this.sysInfo.iscv2xplatform == '1') && this.faceEnable)
                    tip = this.$t('faceDisableTips');
            }
            return tip;
        },
        change_fisheye_track () {
            if (!this.isFisheye)
                return ;
            if (this.fish_support_auto_track(this.fishInstall, this.fishDisplay) == 0)
                return ;
            
            let url, enable;
            if (this.trackEnable && this.showTrack) {
                enable = 0;
                url = '&trackenable=0&showtrack=0';
            } else {
                enable = 1;
                url = '&trackenable=1&showtrack=1';
            }

            do {
                if (enable == 1 && (this.smartEventEnable || this.ptzMotionEnable || this.lprEnable)) {
                    if (this.smartEventEnable &&
                        this.fish_support_vca(this.fishCorrect, this.curDisplay, this.fishInstall) == 0)
                        break;
                    let tip = this.get_auto_track_tips();
                    confirmBox(tip, () => {
                        // disable 互斥功能 移至后端实现,由于权限关系
                        // 比如无事件权限的用户开启auto track，会导致事件disable失败
                        // this.auto_track_disable_other_event(this.smartEventEnable, this.ptzMotionEnable,
                        //     this.lprEnable);
                        this.smartEventEnable = 0;
                        this.ptzMotionEnable = 0;
                        this.lprEnable = 0;
                        this.trackEnable = enable;
                        this.showTrack = enable;
                        this.show_auto_track();
                        set_page_data('autotracking', url);
                    });
                    return ;
                }
            // eslint-disable-next-line no-constant-condition
            } while (0);
            this.trackEnable = enable;
            this.showTrack = enable;
            this.show_auto_track();
            set_page_data('autotracking', url);
            return ;
        },
        change_track () {
            if (this.sysInfo.isfisheye == '1') {
                this.change_fisheye_track();
                return ;
            }
            if (this.sysInfo.iscv2xplatform == '1')
                return ;
            
            if ((this.sysInfo.isspeed == '1' || this.sysInfo.isptzbullet == '1' || this.sysInfo.isminiptzdome == '1')
                && this.trackEnable) {
                this.manualTrack = 1 - this.manualTrack;
                set_manual_track(this.manualTrack);
                let url = `&trackmanualenable=${this.manualTrack}`;
                if (this.manualTrack) {
                    url += '&pos3denable=0';
                    this.posEnable = 0;
                }
                playerApi.MsSetParam('platform', 1);
                playerApi.MsSet3dPosRegion(this.manualTrack, "0:0:0:0");
                send_vb_htm(url);
            } else {
                tipsBox(this.$i18n.t('pleaseSetTrack'));
            }
        },
        update_defog_status () {
            let url = 'paratest=defogstatus';
            send_vb_htm(url).then( res => {
                let txt = res;
                let str = 'OK defogstatus=';
                let ix = txt.indexOf(str);
                if (ix >= 0) {
                    let val = parseInt(txt.substr(ix + str.length, 1));
                    if (val == 0) {
                        console.log('this.manualDefog = 0');
                        this.manualDefog = 0;
                        clearTimeout(this.defogTimer);
                        this.defogTimer = null;
                        return ;
                    }
                }
                this.defogTimer = setTimeout( () => {
                    this.update_defog_status();
                }, 1000);
            })
        },
        change_defog () {
            if (this.sysInfo.isspeed == '0' && this.sysInfo.isminiptzdome == '0' && this.sysInfo.isptzbulletv3 == '0')
                return ;
            let url = '&switchfan=';
            this.manualDefog = 1 - this.manualDefog;
            console.log('this.manualDefog', this.manualDefog);
            if (this.manualDefog) {
                clearTimeout(this.defogTimer);
                this.defogTimer = null;
                this.defogTimer = setTimeout( () => {
                    this.update_defog_status();
                }, 1000);
            }
            url += this.manualDefog;
            send_vb_htm(url);
        },
        // lpr
        get_lpr_name (time, lprData) {
            let jpgName = "";
            let separator = '_';
            if(this.lprSeparator == "0")
                separator = '-';
            else if(this.lprSeparator == "2")
                separator = ' ';
            for(let i=0; i<this.lprSnapSort.length; i++) {
                switch(parseInt(this.lprSnapSort[i])) {
                    case 0:
                        jpgName += time;
                        break;
                    case 1:
                        if (this.isIran && i != 0) {
                            jpgName = jpgName.substring(0, jpgName.length - 1);
                            jpgName += ' ' + separator + lprData.msg;
                        } else
                            jpgName += lprData.msg;
                        break;
                    case 2:
                        if(lprData.type == 1)
                            jpgName += "Black";
                        else if(lprData.type == 2)
                            jpgName += "White";
                        else if(lprData.type == 3)
                            jpgName += "Visitor";
                        break;
                    case 3:
                        if(lprData.speed == 0)
                            jpgName += "#";
                        else
                            jpgName += lprData.speed;
                        break;
                    case 4:
                        if(lprData.direction == 0)
                            jpgName += "#";
                        else if(lprData.direction == 1)
                            jpgName += "Approach";
                        else if(lprData.direction == 2)
                            jpgName += "Away";
                        break;
                    case 5:
                        if(lprData.region < 1 || lprData.region > 4)
                            jpgName += "#";
                        else
                            jpgName += lprData.region;
                        break;
                    case 6:
                        jpgName += this.lprSnapPos;
                        break;
                    case 7:
                        jpgName += this.deviceName;
                        break;
                    case 8:
                        jpgName += this.lprSnapId;
                        break;
                }
                if(i == this.lprSnapSort.length - 1)
                    ;// jpgName += ".jpg";
                else
                    jpgName += separator;
            }
            let trans_name = jpgName.replace(/ /g, " ")
            console.log(jpgName);
            return jpgName;
        },
        get_base_64 (img) {
            let canvas = document.createElement("canvas");
            // console.log('img.naturalWidth', img.naturalWidth, img.naturalHeight);
            canvas.width = img.naturalWidth;
            canvas.height = img.naturalHeight;

            let ctx = canvas.getContext("2d");
            ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
            var dataURL = canvas.toDataURL('image/jpeg');
            canvas = null;  // 需要释放，否则泄漏
            return dataURL;
        },
        base64_to_blob (b64Data, contentType, sliceSize = 512) {
            let byteCharacters = atob(b64Data);
            let byteArr = [];
            for (let offset = 0; offset < byteCharacters.length; offset += sliceSize) {
                let slice = byteCharacters.slice(offset, offset + sliceSize);
                let byteNum = [];
                for (let i=0; i<slice.length; i++) {
                    byteNum.push(slice.charCodeAt(i));
                }
                byteArr.push(new Uint8Array(byteNum));
            }
            let res = new Blob (byteArr, {
                type: contentType
            });
            res = Object.assign(res, {
                preview: URL.createObjectURL(res),
                name: 'XXX.jpg'
            });
            return res;
        },
        plate_img_load (index, row) {
            console.log('plate_img_load');
            if (index != 1)
                return ;
            // window.URL.revokeObjectURL(row.imgSrc);
            if (browser_IE && playerApi.MsGetParam("GetIEauthority") == "0") {
                let img = document.getElementById(`plate_img${index}`);
                let imgData = this.get_base_64(img).split(',')[1];
                let filename = `${this.get_lpr_name(this.lprFilePrefix.substring(0, this.lprFilePrefix.length - 2),
                    this.lastLprData)}.jpg`;
                let fileDate = row.time.split(' ')[0];
                playerApi.MsSaveLPRJpg(filename, imgData, imgData.length, fileDate);
            }
        },
        add_lpr_row (lprData) {
            this.lprLastestType = lprData.type;
            if (this.lprData.length >= this.LPR_MAX_ROW) {
                let rows = document.getElementById("lpr_list").rows.length;
                if (document.getElementById("lprimg_"+this.lprData[this.lprData.length-1].index)) {
                    if (browser_IE) {
                        // 释放小图src，好像没效果，后续版本跟进
                        document.getElementById("lprimg_"+this.lprData[this.lprData.length-1].index).src = null;
                        document.getElementById("lprimg_"+this.lprData[this.lprData.length-1].index).removeNode(true);
                    }
                }
                document.getElementById("lpr_list").deleteRow(rows - 2);
                this.lprData[this.lprData.length-1] = null;
                this.lprData.pop();
            }
            this.lastLprData = lprData;
            let obj = {};
            obj.index = ++this.lprLogsCnt;
            obj.plate = lprData.msg;
            obj.type = this.lprType[lprData.type];
            obj.color = this.lprColor[lprData.platecolor];
            obj.vType = this.vehicleType[lprData.vehicletype];
            obj.vColor = this.lprColor[lprData.vehiclecolor];
            let speed = lprData.speed ? `${lprData.speed}Km/h` : '-';
            if (this.sysInfo.radarsupport == '1' && lprData.direction != 0 && speed == '-')
                speed = '0Km/h';        // has direction,means has speed
            obj.speed = speed;
            obj.direction = this.lprDirec[lprData.direction];
            obj.region = lprData.region == '-1' ? '-' : lprData.region;
            obj.time = lprData.time;
            obj.fileDate = lprData.time.split(' ')[0];
            let filePrefix = lprData.path.substring(lprData.path.indexOf('/LPR/') + 5, lprData.path.length - 4);
            this.lprFilePrefix = filePrefix;
            obj.filename = `4${this.get_lpr_name(filePrefix.substring(0, filePrefix.length - 2), lprData)}`;
            // this.lprInfo = obj;  // 内存泄漏
            document.getElementById('lprplate').innerHTML = obj.plate;
            document.getElementById('lprtype').innerHTML = obj.type;
            document.getElementById('lprcolor').innerHTML = obj.color;
            document.getElementById('lprvtype').innerHTML = obj.vType;
            document.getElementById('lprvcolor').innerHTML = obj.vColor;
            document.getElementById('lprspeed').innerHTML = obj.speed;
            document.getElementById('lprdir').innerHTML = obj.direction;


            obj.imgSrc = `/LPR/${filePrefix}.jpg`;
            let img = new Image();
            img.src = `/LPR/${filePrefix}.jpg`;
            this.lprData.unshift(obj);

            img.onload = () => {
                let base64 = this.get_base_64(img).split(',')[1];
                let blob = this.base64_to_blob(base64, 'image/jpeg');

                let table = document.getElementById("lpr_list");
                // console.log("row count:"+table.rows.length);
                let oneRow = table.insertRow(0);
                oneRow.className = 'el-table__row'
                let cells = new Array(12);
                for (let i = 0; i < 12; i++) {
                    cells[i] = oneRow.insertCell();//单单插入一行是不管用的，需要插入单元格  
                }
                cells[0].title = obj.index;
                // 序号预处理
                if (obj.index.length > 5) {
                    obj.index = obj.index.slice(0, 4) + '...'
                }
                cells[0].innerHTML = obj.index;
                cells[1].innerHTML = obj.plate;
                let imgStr = "<img src=" + blob.preview  + " id='lprimg_"+ obj.index +"'></img>";
                cells[2].innerHTML = imgStr;
                cells[3].innerHTML = obj.type;
                cells[4].innerHTML = obj.color;
                cells[5].innerHTML = obj.vType;
                cells[6].innerHTML = obj.vColor;
                cells[7].innerHTML = obj.speed;
                cells[8].innerHTML = obj.direction;
                cells[9].innerHTML = obj.region;
                cells[10].innerHTML = obj.time;
                let operationHtml = "<a class='icon-search' href='javascript:show_lpr_img(" + obj.index + ")'>";
                let listClass = '';
                if (obj.type != this.lprType[3]) {
                    listClass = 'lpr-add-disable';
                }
                operationHtml += "<a class='icon-list " + listClass + "' href='javascript:show_lpr_list(" + obj.index + ")'>";
                cells[11].innerHTML = operationHtml;

                if (browser_IE && playerApi.MsGetParam("GetIEauthority") == "0") {
                    let filename = `${this.get_lpr_name(this.lprFilePrefix.substring(0, this.lprFilePrefix.length - 2),
                        this.lastLprData)}.jpg`;
                    playerApi.MsSaveLPRJpg(filename, base64, base64.length, obj.fileDate);
                }
                
                img.onload = null;
                img = null;
                cells = null;
                blob = null;
                base64 = null;
            };
            // img.onerror = ()=>{

            // }
        },
        dialog_img_load () {
            window.URL.revokeObjectURL(this.diaInfo.imgSrc);
        },
        show_lpr_img (num) {
            let index = 0;
            for (let i = 0; i < this.lprData.length; i++) {
                if (this.lprData[i].index == num) {
                    index = i;
                    break;
                }
            }
            this.diaInfo.imgSrc = window.URL.createObjectURL(this.lprData[index].eventData);
            this.showDialogForm = false;
            this.showDialog = true;
        },
        show_lpr_list (num) {
            let index = 0;
            for (let i = 0; i < this.lprData.length; i++) {
                if (this.lprData[i].index == num) {
                    index = i;
                    break;
                }
            }
            if (this.lprData[index].type == this.lprType[1] || this.lprData[index].type == this.lprType[2] ||
                 !this.hasLprPermission)
                return ;
            if (!this.hasLprPermission) {
                return;
            }
            this.diaInfo.plate = this.lprData[index].plate;
            this.diaInfo.type = 1;
            this.showDialogForm = true;
            this.showDialog = true;
        },
        get_add_status (txt) {
            let ix = txt.indexOf('OK');
            let plate = this.diaInfo.plate.replace(/\s+/g,'');
            if (ix >= 0) {
                show_save_successful();
            } else {
                ix = txt.indexOf('exist');
                if (ix >= 0) {
                    msgBox(this.$t('saveSamePlate'), 'error');
                }
                ix = txt.indexOf('upmax');
                if (ix >= 0)
                    tipsBox(this.$t('outOfCapacity'));
            }
        },
        add_lpr_list () {
            let plate = this.diaInfo.plate.replace(/\s+/g,'');
            let url = `&type=${this.diaInfo.type}&plate=${encodeURIComponent(plate)}&validtype=0&note=`;
            set_page_data('addlprlist', url).then( res => {
                this.get_add_status(res);
            });
            this.showDialog = false;
        },
        getValueFromOri(type, info, start, nmemb) {
            let value;
            switch(type) {
                case "ms_int":
                    var temp=new Int32Array(info, start, nmemb);
                    value = parseInt(temp[0], 10);
                    break;
                case "ms_longlong":
                    value = new Uint32Array(event.data, start, nmemb);
                    //value = parseInt((new Int32Array(info, start, nmemb)), 10);
                    break;
                case "ms_char":
                    value = "";
                    var item = new Uint8Array(info, start, nmemb);
                    value = Utf8ArrayToStr(item)
                    break;
                default:
                    break;
            }
            return value;
        },
        lpr_msg_cb (event) {
            if (event.data === 'pong')
                this.webSocketObject.pingPong = 'pong';
            if (typeof event.data === 'string')
                this.add_lpr_row(JSON.parse(event.data));
            // if (event.data instanceof ArrayBuffer)
            //     let buffer = event.data;
            if (event.data instanceof Blob) {

                let reader = new FileReader();
                reader.readAsArrayBuffer(event.data);
                let that = this;
                reader.onload = function() {
                    let array = this.result;
     
                    let offset = 0;
                    let oLprStr = new Object();
                    offset = 0;
                    oLprStr.smallFileSize = that.getValueFromOri("ms_int", array, offset, 1);
                    offset += 4;
                    oLprStr.bigFileSize = that.getValueFromOri("ms_int", array, offset, 1);
                    offset += 4;

                    let smallFileOffset = 4 + 4;
                    let bigFileOffset = smallFileOffset + oLprStr.smallFileSize;
                    let img = document.getElementById('plate_bigimg');
                    // img.onload = function (e) {
                    //     window.URL.revokeObjectURL(img.src); // Clean up after yourself.
                    // }
                    window.URL.revokeObjectURL(img.src); // Clean up after yourself.
                    // window.URL.revokeObjectURL(that.plateImgSrc);
                    


                    let filename = that.lprData[0].filename;
                    let fileDate = that.lprData[0].fileDate;
                    let imgArray = new Uint8Array(array, bigFileOffset, oLprStr.bigFileSize);
                    let mime = "image/jpeg";
                    let imgBlob = new Blob([imgArray], { type: mime });
                    // that.plateImgSrc = window.URL.createObjectURL(imgBlob);
                    document.getElementById('plate_bigimg').src = window.URL.createObjectURL(imgBlob);
                    that.lprData[0].eventData = imgBlob;
                    if (browser_IE && playerApi.MsGetParam("GetIEauthority") == "0") {
                        img.onload = () => {
                            let imgData = that.get_base_64(img).split(',')[1];
                            let jpgName = `${filename}.jpg`;
                            playerApi.MsSaveLPRJpg(jpgName, imgData, imgData.length, fileDate);

                            img.onload = null;
                            img = null;
                        }
                    } else {
                        img = null;
                    }
                    reader.onload = null;
                    reader = null;
                    oLprStr = null;
                    imgArray = null;
                    imgBlob = null;
                };
            }
        },
        lpr_err (event) {
            if (this.lprSocket)
                this.lprSocket.close();
        },
        lpr_connect () {
            let wsUrl = 'ws://';
            if (window.location.protocol == "https:")
                wsUrl = 'wss://';
            wsUrl += `${window.location.host}/webstream/lpr_liveview`;
            if (CheckBrowserType().type == 'Safari')
                wsUrl += `?${ASTGUI._getClientDigest(wsUrl)}`;
            this.lprSocket = this.webSocketObject(wsUrl, this.lpr_msg_cb, this.lpr_err);
        },
        change_lpr_mode () {
            if (browser_IE && ASTGUI.cookies.getCookie("authtips") == "1") {
                setTimeout( () => {
                    if (playerApi.MsGetParam("GetIEauthority") == '1' && !this.hasDestroy)
                        tipsBox(this.$t('lprAdminPrivTips'),'warning');
                }, 2000);
                ASTGUI.cookies.setCookie("authtips", "0");
            }
            if (this.lprStatus == 1)
                this.lprSel = ASTGUI.cookies.getCookie("lprType") ? parseInt(ASTGUI.cookies.getCookie("lprType")) : 0;
            if (this.isLprMode) {
                playerApi.MsSetParam("isLPR", "0");
                if (document.getElementsByClassName('selCanvas').length)
                    document.getElementsByClassName('selCanvas')[0].display = 'none';
                ASTGUI.cookies.setCookie("lprmode", "0");
            } else {
                playerApi.MsSetParam("isLPR", "1");
                ASTGUI.cookies.setCookie("lprmode", "1");
                if (document.getElementsByClassName('selCanvas').length)
                    document.getElementsByClassName('selCanvas')[0].display = '';
                if (this.lprSocket == null) {
                    this.lpr_connect();
                }
            }
            this.isLprMode = !this.isLprMode;
            this.lprLayTimer = setTimeout( () => {
                this.change_lpr_lays(this.lprSel);
            }, 1000);
        },
        is_rec_or_limit() {
            // if(this.limitingstatus)
            //     return 1;
            if(this.sysInfo.isptzbullet == '1' && this.mirctrl > 1)
                return 1;
            for (let i = 0; i < this.PATTERN_NUM; i++) {
                if (this.patternArr[i].record) {
                    return 1;
                }
            }
            return 0;
        },
        click_auto_scan () {
            if (playerApi.MsGetParam("fisheyeCorrectionModel") == '1') {
                if (this.autoScanStatus == 1) {
                    playerApi.MsSetParam("FisheyeSoftwareCorrectionCmd", 0);
                    this.autoScanStatus = 0;
                } else if (this.autoScanStatus == 0) {
                    playerApi.MsSetParam("FisheyeSoftwareCorrectionCmd", 13);
                    this.autoScanStatus = 1;
                } else {
                    playerApi.MsSetParam("FisheyeSoftwareCorrectionCmd", 13);
                }
                return ;
            }
            if (this.is_rec_or_limit()) {
                return;
            }
            if (this.isFisheye) {
                this.send_fish_ptz(13);
                return ;
            }        
            this.autoScanStatus = !this.autoScanStatus;
            if (this.autoScanStatus) {
                this.$store.commit('app/SET_AUTOSCAN', true)
                send_ptz_cmd_to_device(13);
            }
            else
                send_ptz_cmd_to_device(0);
        },
        is_region (stream) {
            if (this.fishDisplay == fishDisplayMode.DISPLAY_1O || this.fishDisplay == fishDisplayMode.DISPLAY_180P ||
                this.fishDisplay == fishDisplayMode.DISPLAY_360P)
                return 0;
            else if (this.fishDisplay == fishDisplayMode.DISPLAY_1O3R ||
                this.fishDisplay == fishDisplayMode.DISPLAY_1P3R) {
                if (stream == 0)
                    return 0;
            } else if (this.fishDisplay == fishDisplayMode.DISPLAY_1O1P3R) {
                if (stream == 0 || stream == 1)
                    return 0;
            }
            return 1;
        },
        send_fish_ptz (cmd) {
            let channel = this.get_channel_index();
            if (channel < 0 || channel > 4)
                return ;
            let url = '&ipncptz=';
            if (cmd == 13) {
                if (this.fishAutoScan[channel] == 1) {
                    this.send_ptz_stop_cmd();
                    this.fishAutoScan[channel] = 0;
                    this.autoScanStatus = 0;
                    return ;
                } else {
                    if (this.is_region(channel) == 1) {
                        this.autoScanStatus = 1;
                        this.fishAutoScan[channel] = 1;
                    } else
                        return ;
                }
            } else {
                this.autoScanStatus = 0;
                this.fishAutoScan[channel] = 0;
            }

            switch (cmd) {
                case 7:
                    url += 'fishptzleft=';
                    break;
                case 12:
                    url += 'fishptzright=';
                    break;
                case 5:
                    url += 'fishptzup=';
                    break;
                case 6:
                    url += 'fishptzdown=';
                    break;
                case 8:
                    url += 'fishptztopleft=';
                    break;
                case 9:
                    url += 'fishptztopright=';
                    break;
                case 10:
                    url += 'fishptzleftdown=';
                    break;
                case 11:
                    url += 'fishptzrightdown=';
                    break;
                case 3:
                    url += 'zoomin=';
                    this.autoScanStatus = 0;
                    break;
                case 4:
                    url += 'zoomout=';
                    this.autoScanStatus = 0;
                    break;
                case 13:
                    url += 'fishAuto=';
                    break;
            }
            url += `${channel}:0;`;
            send_vb_htm(url);
            if (cmd != 13)
                this.ptzIsUsing = 1;
            if (cmd != 13)
                this.update_digital_label(1);
            else
                this.update_digital_label(0, 1);
        },
        reset_patrol_img (val) {
            if (this.patrolRecoveryEnable == 1 && this.sysInfo.isptzbullet == '1' && val == 0)
                return ;
            this.patrolMode = 0;
        },
        send_ptz_stop_cmd () {
            let correct = playerApi.MsGetParam("fisheyeCorrectionModel");
            if (correct == '1') {
                playerApi.MsSetParam("FisheyeSoftwareCorrectionCmd", "0");
                return ;
            }
            let str = '&ipncptz=stop';
            if (this.isFisheye) {
                //stop get fish ptz
                this.ptzTimer = 0;
                clearTimeout(this.ptzTimer);
                this.ptzTimer = setTimeout( () => {
                    this.get_fish_ptz_region();
                }, 1000);
                playerApi.MsSetParam("ptzoperation", 0);
                let channel = this.get_channel_index();
                if (channel < 0 || channel > 4)
                    return ;
                else
                    str += `=${channel}`;
            }
            send_vb_htm(str);
            this.ptzIsUsing = 0;
        },
        ptz_mouse_out () {
            if (this.ptzIsUsing)
                this.send_ptz_stop_cmd();
        },
        send_ptz_cmd (cmd) {
            let correct = playerApi.MsGetParam("fisheyeCorrectionModel");
            if (correct == '1') {
                if (this.isSupportPtzCtrl) {
                    if (cmd == 13 && this.autoScanStatus) {
                        playerApi.MsSetParam("FisheyeSoftwareCorrectionCmd", 0);
                        this.autoScanStatus = false;
                    } else if (cmd == 13 && !this.autoScanStatus) {
                        playerApi.MsSetParam("FisheyeSoftwareCorrectionCmd", cmd);
                        this.autoScanStatus = true;
                    } else {
                        playerApi.MsSetParam("FisheyeSoftwareCorrectionCmd", cmd);
                    }
                }
                return ;
            }
            if (this.sysInfo.isfisheye == '1') {
                this.send_fish_ptz(cmd);
                return ;
            }
            if (cmd == 13) {                   
                if (this.autoScanStatus) {
                    this.send_ptz_stop_cmd();
                    this.autoScanStatus = false;
                }
            } else
                this.autoScanStatus = false;
            if ((cmd == 1 || cmd == 2) && this.sysInfo.islensnormal == '1' && this.ptzSpeed != 1) {
                send_vb_htm('&ptzspeed=0');
            }
            if (this.sysInfo.radarsupport == '1' && this.radarEnable && this.firstRadarTips &&
                (cmd == 3 || cmd == 4 || cmd == 18)) {
                this.firstRadarTips = false;
                show_warning_msg(this.$t("zoomChangedNote"));
            }
            send_ptz_cmd_to_device(cmd);
            if (cmd != 13)
                this.ptzIsUsing = 1;
        },
        stop_ptz_status () {
            this.playTour = 0;
            this.playPattern = 0;
            for (let i=0; i<this.PATTERN_NUM; i++) {
                this.patternArr[i].record = 0;
            }
            this.autoScanStatus = 0;
            this.patrolMode = 0;
        },
        format_preset_id (i, bits) {
            if (bits == 3) {
                if (i > 0 && i < 10)
                    return `00${i}`;
                else if (i >= 10 && i < 100)
                    return `0${i}`;
                else
                    return i;
            } else if (bits == 2) {
                if (i > 0 && i < 10)
                    return `0${i}`;
                else
                    return i;
            }
        },
        get_quick_preset_name (index) {
            if (index == ptzQuick.FLIP_PRESET)
                return this.$t('autoFlip');
            else if (index == ptzQuick.GOTO_ZERO_PRESET)
                return this.$t('gotoZero');
            else if (index == ptzQuick.SELF_CHECK_PRESET)
                return this.$t('selfCheck');
            else if (index == ptzQuick.PATROL_PRESET)
                return this.$t('patrol');
            else if (index >= ptzQuick.TOUR1_PRESET && index <= ptzQuick.TOUR8_PRESET)
                return `${this.$t('path')}${index - ptzQuick.TOUR1_PRESET + 1}`;
            else if (index >= ptzQuick.PATTERN1_PRESET && index <= ptzQuick.PATTERN4_PRESET)
                return `${this.$t('pattern')}${index - ptzQuick.PATTERN1_PRESET + 1}`;
            else if (index == ptzQuick.STOP_SCAN_PRESET)
                return this.$t('stopScan');
            else if (index == ptzQuick.AUTO_SCAN_PRESET)
                return this.$t('autoScan');
            else if (index == ptzQuick.VERTI_SCAN_PRESET)
                return this.$t('tiltScan');
            else if (index == ptzQuick.FULLVIEW_SCAN_PRESET)
                return this.$t('panoramaScan');
        },
        
        init_preset () {
            this.presetTable = [];
            for (let i=1; i<=this.PRESET_NUM; i++) {
                let obj = {};
                if (this.sysInfo.isfisheye == '0' && i >= ptzQuick.PRESET_START && i <= ptzQuick.PRESET_END) {
                    obj.quick = 1;
                    obj.name = this.get_quick_preset_name(i);
                    obj.exist = 1;
                    if (this.sysInfo.isptzbullet == '1' && i == ptzQuick.FLIP_PRESET)
                        obj.exist = 0;
                    if (this.sysInfo.isnewspeed == '0' && this.sysInfo.isminiptzdome == '0' &&
                        (i == ptzQuick.VERTI_SCAN_PRESET || i == ptzQuick.FULLVIEW_SCAN_PRESET))
                        obj.exist = 0;
                    if (this.sysInfo.isptzdevicewithouttour == '1' && i == ptzQuick.PATROL_PRESET)
                        obj.exist = 0;
                } else {
                    obj.quick = 0;
                    let index = this.presetArr.indexOf(i);
                    if (index != -1) {
                        obj.exist = 1;
                        obj.name = this.presetName[index];
                    } else {
                        obj.exist = 0;
                        obj.name = `${this.$t('preset')} ${i}`;
                    }
                }
                this.presetTable.push(obj);
            }
        },
        init_fish_preset (channel) {
            this.presetTable = [];
            for (let i=1; i<=this.PRESET_NUM; i++) {
                let obj = {};
                obj.quick = 0;
                let index = this.fishPresetArr[channel].indexOf(i);
                if (index != -1) {
                    obj.exist = 1;
                    obj.name = this.fishPresetName[channel][index];
                } else {
                    obj.exist = 0;
                    obj.name = `${this.$t('preset')} ${i}`;
                }
                this.presetTable.push(obj);
            }
        },
        click_preset_label (index) {
            if (index >= ptzQuick.PRESET_START-1 && index <= ptzQuick.PRESET_END-1) {
                return ;
            }
            if (this.displayDisablePtz || !this.isSupportPtzSettings)
                return ;
            this.editPreset = index;
            this.$nextTick().then( ()=> {
                this.$refs[`preset${index}`][0].$el.children[0].focus();
                this.$refs[`preset${index}`][0].$el.children[0].select();
            });
        },
        presetInputBlur (index) {
            this.editPreset = -1;
            if (this.presetTable[index].name.trim() == '') {
                this.presetTable[index].name = `${this.$t('preset')} ${index+1}`;
            }
        },
        save_preset (index) {
            index = index + 1;
            this.editPreset = -1;
            let ptzData;
            if (this.sysInfo.isfisheye == '1' && is_fisheye_software_correction()) {
                ptzData = playerApi.MsGetParam("FisheyeSoftwareCorrectionPtzRegion");
                if (ptzData == '')
                    return ;
            }
            let des = this.presetTable[index - 1].name;
            if (get_length(des) > 20)
                des = sub_string_x(des, 20);
            if (!this.isFisheye) {
                let flag = 0;
                let i = this.presetArr.indexOf(index);
                if (i != -1) {
                    this.presetName[i] = des;
                    flag = 1;
                }
                if (flag != 1) {
                    this.presetArr.push(index);
                    this.presetName.push(des);
                }
            }
            this.presetTable[index - 1].name = des;
            this.presetTable[index - 1].exist = 1;

            let url = '';
            if (this.sysInfo.isfisheye == '1') {
                if (is_fisheye_software_correction())
                    url = `setfishptzpreset=1:${index}:${ptzData}:${encodeURIComponent(des)}`;
                else
                    url = `&ipncptz=setfishpreset=${this.get_channel_index()}:${index}:${encodeURIComponent(des)}`;
                this.sync_fish_preset();
                playerApi.MsSetParam("fisheyeSoftwarePresetChanged", index);
            } else
                url = `&ipncptz=setpreset=${index}:${encodeURIComponent(des)}`;
            send_vb_htm(url);
        },
        del_preset (index) {
            index = index + 1;
            let des = `${this.$t('preset')} ${index}`;
            let i = this.presetArr.indexOf(index);
            if (i != -1) {
                this.presetArr.splice(i, 1);
                this.presetName.splice(i, 1);
            }
            this.presetTable[index - 1].name = des;
            this.presetTable[index -1].exist = 0;
            if (this.watchId == index) {
                this.watchEnable = false;
			}
            let url;
            if (this.sysInfo.isfisheye == '1') {
                url = `&ipncptz=clearfishpreset=${this.get_channel_index()}:${index}`;
                this.sync_fish_preset();
                playerApi.MsSetParam("fisheyeSoftwarePresetChanged", index);
            } else
                url = `&ipncptz=clearpreset=${index}`;
            send_vb_htm(url);
            this.notify_del_preset(index);
        },
        goto_preset (val) {
            val = val + 1;
            if (this.sysInfo.isfisheye == '0') {
                if (val >= ptzQuick.SELF_CHECK_PRESET && val <= ptzQuick.PRESET_END) {
                    if (this.is_rec_or_limit())
                        return ;
                }
                if (val < ptzQuick.TOUR1_PRESET || val > ptzQuick.PATTERN4_PRESET) {
                    this.autoScanStatus = true;
                }
                if (val == ptzQuick.AUTO_SCAN_PRESET) {
                    //
                    setTimeout( () => {
                        this.show_auto_limit_tip();
                    }, 500);
                } else if (val == ptzQuick.PATROL_PRESET) {
                    this.autoScanStatus = false;
                } else if (val >= ptzQuick.TOUR1_PRESET && val <= ptzQuick.TOUR8_PRESET) {
                    this.stop_ptz_status();
                    let id = val - ptzQuick.TOUR1_PRESET;
                    this.playTour = id + 1;
                } else if (val >= ptzQuick.PATTERN1_PRESET && val <= ptzQuick.PATTERN4_PRESET) {
                    this.stop_ptz_status();
                    let id = val - ptzQuick.PATTERN1_PRESET;
                    if (this.patternArr[id].exist) {
                        //this.playPattern = id + 1;
                        clearTimeout(this.patternTimer);
                        this.patternTimer = setTimeout( () => {
                            this.update_pattern_status();
                        }, 300);
                    }
                } else
                    this.autoScanStatus = false;
            } else {
                if("1" == ASTGUI.cookies.getCookie("fisheyeCorrectionModel") ||
                    "1" == playerApi.MsGetParam("fisheyeCorrectionModel")) {
                    let url = `&getfishptzpreset=1:${val}`;
                    send_vb_htm(url).then( res => {
                        let txt = res;
                        let ix = txt.indexOf('OK getfishptzpreset=');
                        if (ix >= 0) {
                            let code = txt.substring(ix + 20);
                            playerApi.MsSetParam("FisheyeSoftwareCorrectionPtzRegion", code);
                        }
                    });
                    this.autoScanStatus = false;
                    return ;
                }
                this.autoScanStatus = false;
            }
            let url;
            if (this.sysInfo.isfisheye == '1') {
                url = `&ipncptz=gotofishpreset=${this.get_channel_index()};`;
                // UpdateDigitalLabel(0);
            } else
                url = `&ipncptz=gotopreset=`;
            url += val;
            send_vb_htm(url);
            this.notify_update_preset(val);
        },
        notify_del_preset (preset) {
            if (this.isLpr) {
                if (this.curPresetPos == preset)
                    this.curPresetPos = -1;
                if (preset <= 4 && preset >= 1) {
                    this.lprRoiRegionPreset[preset] = ['0:0:0:0', '0:0:0:0', '0:0:0:0', '0:0:0:0'];
                }
                if (this.lprSel && !this.zoomEnable) {
                    this.destroy_lpr_roi();
                    this.draw_lpr_roi();
                }
            }
        },
        notify_update_preset (preset) {
            if (this.isLpr) {
                this.curPresetPos = preset;
                if (this.lprSel && !this.zoomEnable) {
                    this.destroy_lpr_roi();
                    this.draw_lpr_roi();
                }
            }
        },
        init_tour () {
            this.tourArr = this.tourList;
        },
        get_channel_index () {
            if (this.is_fish_software_correct())
                return 1;
            let index = 0;
            if (this.fishCorrect == fishCorrectMode.JOINT_VIDEO) {
                if (this.playUrlIndex[0] != -1) {
                    if (this.fishDisplay == fishDisplayMode.DISPLAY_4R ||
                        this.fishDisplay == fishDisplayMode.DISPLAY_1O3R ||
                        this.fishDisplay == fishDisplayMode.DISPLAY_1P3R)
                        index = this.fishCurWnd;
                    else
                        index = 0;
                } else
                    index = -1;
            } else
                index = this.playUrlIndex[this.fishCurWnd];
            return index;
        },
        sync_fish_preset () {
            this.fishPresetArr[this.get_channel_index()] = [];
            this.fishPresetName[this.get_channel_index()] = [];
            for (let i=0; i<this.presetTable.length; i++) {
                if (this.presetTable[i].exist) {
                    this.fishPresetArr[this.get_channel_index()].push(i + 1);
                    this.fishPresetName[this.get_channel_index()].push(this.presetTable[i].name);
                }
            }
        },
        sync_fish_tour () {
            this.fishTourList[this.get_channel_index()] = [...this.tourArr];
        },
        get_valid_preset_pos() {
            for (const pos of this.presetOptions) {
                let find = false
                for (const item of this.editTour) {
                    if (item.pid == pos) {
                        find = true
                        break
                    }
                }
                if (!find) {
                    return pos
                }
            }
            return 1
        },
        add_tour_content () {
            if (this.editTour.length == this.TOUR_MAX_PRESET_NUM)
                return ;
            let obj = {};
            obj.pid = this.get_valid_preset_pos();
            obj.speed = 30;
            if (this.sysInfo.isptzbullet == '1')
                obj.time = '15';
            else
                obj.time = '10';
            this.editTour.push(obj);
        },
        del_tour_content (index) {
            this.editTour.splice(index, 1);
            if (this.selectedPid >= this.editTour.length)
                this.selectedPid = -1;
        },
        up_tour_content () {
            if (this.selectedPid) {
                let t = this.editTour[this.selectedPid];
                this.editTour[this.selectedPid] = this.editTour[this.selectedPid - 1];
                this.editTour[this.selectedPid - 1] = t;
                this.selectedPid--;
            }
        },
        dw_tour_content () {
            if (this.selectedPid == -1)
                return;
            if (this.selectedPid != this.editTour.length - 1) {
                let t = this.editTour[this.selectedPid];
                this.editTour[this.selectedPid] = this.editTour[this.selectedPid + 1];
                this.editTour[this.selectedPid + 1] = t;
                this.selectedPid++;
            }
        },
        tourTimeLimit (index) {
        },
        tourTimeBlur (index) {
            for(let i=0; i<this.editTour.length; i++) {
                let time = this.editTour[i].time
                if (this.sysInfo.isptzbullet == '1') {
                    if (!time || parseInt(time) < 15 || parseInt(time) > 120 || /[^0-9]/.test(time)) {
                        this.error = {
                            show: true,
                            id: i,
                            msg: '15 ~ 120'
                        }
                        return
                    }
                }else {
                    if (!time || parseInt(time) < 10 || parseInt(time) > 120 || /[^0-9]/.test(time)) {
                        this.error = {
                            show: true,
                            msg: '10 ~ 120',
                            id: i,
                        }
                        return
                    }

                }
            }
            this.error = {}
        },		
		cancel_save_tour() {
            this.setTour = -1;
            this.error = {};
		},
        save_tour () {
            this.tourArr[this.setTour] = [...this.editTour];
            let id = '', speed = '', time = '';
            for(let i=0; i<this.tourArr[this.setTour].length; i++) {
                if (i) {
                    id += ':';
                    speed += ':';
                    time += ':';
                }
                id += this.tourArr[this.setTour][i].pid;
                speed += this.tourArr[this.setTour][i].speed;
                let curTime = this.tourArr[this.setTour][i].time

                if (this.sysInfo.isptzbullet == '1') {                    
					if (!curTime || parseInt(curTime) < 15 || parseInt(curTime) > 120 || /[^0-9]/.test(curTime)) {
                        this.error = {
                            show: true,
                            id: i,
                            msg: '15 ~ 120'
                        }
                        return
                    }
                } else {
                    if (!curTime || parseInt(curTime) < 10 || parseInt(curTime) > 120 || /[^0-9]/.test(curTime)) {
                        this.error = {
                            show: true,
                            id: i,
                            msg: '10 ~ 120'
                        }
                        return
                    }
				}
                time += curTime;
            }
            let url;
            if (this.isFisheye) {
                url = `&ipncptz=setfishtour=${this.get_channel_index()};${this.setTour + 1};${id};${speed};${time};` +
                    `${this.tourArr[this.setTour].length}`;
                this.sync_fish_tour();
                playerApi.MsSetParam("fisheyeSoftwarePatrolChanged", id);
            } else {
                url = `&ipncptz=settour=${this.setTour + 1};${id};${speed};${time};` +
                    `${this.tourArr[this.setTour].length}`;
            }
            if(this.error.show){
                return
            }
            send_vb_htm(url);
            this.setTour = -1;
        },
        async update_tour_status () {
            let url = 'paratest=playtour';
            let txt = await send_vb_htm(url);
            let str = 'OK playtour=';
            let ix = txt.indexOf(str);
            if (ix >= 0) {
                let val = parseInt(txt.substr(ix + str.length, 1));
                if (val == 0) {
                    if (!this.patrolRecover) {
                        this.tourTimerFlag = false;
                        clearTimeout(this.tourTimer);
                        this.tourTimer = null;
                    }
                    this.playTour = 0;
                } else if (val >= 2) {
                    this.playTour = parseInt(val - 1);
                }
            }
            if (this.tourTimerFlag) {
                this.tourTimer = setTimeout( () => {
                    this.update_tour_status();
                }, 1000);
            }
        },
        send_tour (index) {
            let url = '&ipncptz=', name = 'runtour';
            let callback = null;

            if (this.is_rec_or_limit()) {
                return;
            }

            if (this.isFisheye) {
                if (ASTGUI.cookies.getCookie("fisheyeCorrectionModel") == '1' ||
                    playerApi.MsGetParam("fisheyeCorrectionModel") == '1') {
                    // fisheyesoftware--set and start patrol
                    url = '&getfishptztour=1:';
                    name = '';
                    callback = res => {
                        let txt = res;
                        let ix = txt.indexOf('OK getfishptztour=');
                        if (ix >= 0) {
                            let code = txt.substring(ix + 18);
                            playerApi.MsSetParam("FisheyeSoftwareCorrectionPtzPatrol", `${index};${code}`);
                        }
                    }
                } else {
                    url = `&ipncptz=runfishtour=${this.get_channel_index()};`;
                    name = '';
                    this.update_digital_label(1, 1);
                }
            }
            if (this.playTour == index) {
                name = 'toursp';
                if (this.isFisheye) {
                    if (ASTGUI.cookies.getCookie("fisheyeCorrectionModel") == '1' ||
                        playerApi.MsGetParam("fisheyeCorrectionModel") == '1') {
                        playerApi.MsSetParam("FisheyeSoftwareCorrectionPtzPatrol", '');
                        return ;
                    } else {
                        name = '';
                        url = `&ipncptz=spfishtour=${this.get_channel_index()};`;
                        this.update_digital_label(0, 1);
                    }
                    this.fishPlayTour[this.get_channel_index()] = 0;
                }
                this.playTour = 0;
            } else {
                if (this.isFisheye)
                    this.fishPlayTour[this.get_channel_index()] = index;
                this.playTour = index;
            }
            if (name != '')
                url += `${name}=`;
            url += index;
            send_vb_htm(url).then(callback);

            if (name == 'runtour' && this.tourTimerFlag == false) {
                this.tourTimerFlag = true;
                this.tourTimer = setTimeout( () => {
                    this.update_tour_status();
                }, 300);
            }
        },
        set_tour (index) {
            this.setTour = index - 1;
            this.editTour = [...this.tourArr[this.setTour]];
        },
        del_tour (index) {
            this.tourArr[index - 1] = [];
            this.tourArr = [...this.tourArr];
            let url;
            if (this.isFisheye) {
                url = `&ipncptz=setfishtour=${this.get_channel_index()};${index};;;;0`;
                this.sync_fish_tour();
                playerApi.MsSetParam("fisheyeSoftwarePatrolChanged", index);
            } else
                url = `&ipncptz=settour=${index};;;;0`;
            send_vb_htm(url);
        },
        send_pattern (index) {
            if (this.is_rec_or_limit()) {
                return;
            }
            let url, name;
            if (this.playPattern == index) {
                this.playPattern = 0;
                name = 'patternsp';
            } else {
                this.playPattern = index;
                this.autoScanStatus = false;
                this.playTour = 0;
                name = 'patternrun';
            }
            url = `&ipncptz=${name}=${index}`;
            send_vb_htm(url);
            if (name == 'patternrun') {
                clearTimeout(this.patternTimer);
                this.patternTimer = setTimeout( () => {
                    this.update_pattern_status();
                }, 300);
            }
        },
        record_pattern (index) {
            // if (this.is_rec_or_limit()) {
            //     return;
            // }
            if(this.sysInfo.isptzbullet == '1' && this.mirctrl > 1)
                return;
            let url;
            if (this.patternArr[index - 1].record == 0) {
                for (let i=0; i<this.patternArr.length; i++) {
                    this.patternArr[i].record = 0;
                }
                this.patternArr[index - 1].record = 1;
                this.patternArr[index - 1].exist = 0;
                url = `&ipncptz=recordstart=${index}`;
            } else {
                this.patternArr[index - 1].record = 0;
                this.patternArr[index - 1].exist = 1;
                url = `&ipncptz=recordsp=${index}`;
            }
            this.patternArr = [...this.patternArr];
            this.autoScanStatus = false;
            this.playTour = 0;
            send_vb_htm(url);
        },
        del_pattern (index) {
            this.patternArr[index - 1].record = 0;
            this.patternArr[index - 1].exist = 0;
            this.patternArr = [...this.patternArr];
            if (this.playPattern == index)
                this.playPattern = 0;
            let url = `&ipncptz=patterndelete=${index}`;
            send_vb_htm(url);
        },
        update_pattern_status () {
            let url = 'paratest=playpattern';
            send_vb_htm(url).then( res => {
                let txt = res;
                let str = 'OK playpattern=';
                let ix = txt.indexOf(str);
                if (ix >= 0) {
                    let ret = parseInt(txt.substr(ix + str.length, 1));
                    if (ret == 0) {
                        clearTimeout(this.patternTimer);
                        this.patternTimer = null;
                        this.playPattern = 0;
                        return ;
                    } else if (ret >= 2) {
                        this.playPattern = ret - 1;
                    }
                }
                this.patternTimer = setTimeout( () => {
                    this.update_pattern_status();
                }, 1000);
            });
        },
        show_auto_limit_tip () {
            let url = 'paratest=isshowautoscantip';
            send_vb_htm(url).then( res => {
                let txt = res;
                let tip = 'OK isshowautoscantip=';
                let ix = txt.indexOf(tip);
                if (ix >= 0) {
                    let val = parseInt(txt.substr(ix + tip.length, 1));
                    if (val == 1)
                        playerApi.MsSetParam("setPtzTiltTip", this.$t('ptzLimit360'));
                }
            });
        },
        setPlayerRegionAlarm (alarm) {
            playerApi.MsSetParam("polygonalarm", alarm);
        },
        update_vca_mask(res) {
            let txt = res;
            let region = 0;
            let strLength = 'OK intrusionenterplgx.';
            let presetIx = txt.indexOf(strLength);
            if ( presetIx >= 0) {
                region = parseInt(txt.substr(presetIx + strLength.length, 1));
            }
            if (region > 4 || region < 0) {
                return;
            }
            let vcaData = txt.split("OK ");
            let i = 1, j = 0;
            let argName = `intrusionenterplgx.${region}=`;
            let ix = vcaData[i].indexOf(argName);
            let enterPlgX,enterPlgY,exitPlgX,exitPlgY,admotionPlgX,admotionPlgY;
            let loiterPlgX,loiterPlgY,objectleftPlgX,objectleftPlgY;
            let countLineData = ["", "", "", ""];
            let regionalPplPlgX, regionalPplPlgY;

            if (ix >= 0) {
                let plgTmp = vcaData[i].substr(ix + argName.length,vcaData[i].indexOf("\n"));
                enterPlgX = plgTmp.split(';');
                i++;
            }
            argName = `intrusionenterplgy.${region}=`;
            ix = vcaData[i].indexOf(argName);
            if (ix >= 0) {
                let plgTmp = vcaData[i].substr(ix + argName.length,vcaData[i].indexOf("\n"));
                enterPlgY = plgTmp.split(';');
                i++;
            }

            argName = `intrusionexitplgx.${region}=`;
            ix = vcaData[i].indexOf(argName);
            if (ix >= 0) {
                let plgTmp = vcaData[i].substr(ix + argName.length,vcaData[i].indexOf("\n"));
                exitPlgX = plgTmp.split(';');
                i++;
            }
            argName = `intrusionexitplgy.${region}=`;
            ix = vcaData[i].indexOf(argName);
            if (ix >= 0) {
                let plgTmp = vcaData[i].substr(ix + argName.length,vcaData[i].indexOf("\n"));
                exitPlgY = plgTmp.split(';');
                i++;
            }

            argName = `admotionplgx.${region}=`;
            ix = vcaData[i].indexOf(argName);
            if (ix >= 0) {
                let plgTmp = vcaData[i].substr(ix + argName.length,vcaData[i].indexOf("\n"));
                admotionPlgX = plgTmp.split(';');
                i++;
            }
            argName = `admotionplgy.${region}=`;
            ix = vcaData[i].indexOf(argName);
            if (ix >= 0) {
                let plgTmp = vcaData[i].substr(ix + argName.length,vcaData[i].indexOf("\n"));
                admotionPlgY = plgTmp.split(';');
                i++;
            }

            argName = `loiteringplgx.${region}=`;
            ix = vcaData[i].indexOf(argName);
            if (ix >= 0) {
                let plgTmp = vcaData[i].substr(ix + argName.length,vcaData[i].indexOf("\n"));
                loiterPlgX = plgTmp.split(';');
                i++;
            }
            argName = `loiteringplgy.${region}=`;
            ix = vcaData[i].indexOf(argName);
            if (ix >= 0) {
                let plgTmp = vcaData[i].substr(ix + argName.length,vcaData[i].indexOf("\n"));
                loiterPlgY = plgTmp.split(';');
                i++;
            }

            argName = `objectplgx.${region}=`;
            ix = vcaData[i].indexOf(argName);
            if (ix >= 0) {
                let plgTmp = vcaData[i].substr(ix + argName.length,vcaData[i].indexOf("\n"));
                objectleftPlgX = plgTmp.split(';');
                i++;
            }
            argName = `objectplgy.${region}=`;
            ix = vcaData[i].indexOf(argName);
            if (ix >= 0) {
                let plgTmp = vcaData[i].substr(ix + argName.length,vcaData[i].indexOf("\n"));
                objectleftPlgY = plgTmp.split(';');
                i++;
            }

            argName = `countline.${region}=`;
            ix = vcaData[i].indexOf(argName);
            if (ix >= 0) {
                let plgTmp = vcaData[i].substring(ix + argName.length,vcaData[i].indexOf("\n"));
                countLineData[0] = plgTmp;
                i++;
            }
            argName = `countline1.${region}=`;
            ix = vcaData[i].indexOf(argName);
            if (ix >= 0) {
                let plgTmp = vcaData[i].substring(ix + argName.length,vcaData[i].indexOf("\n"));
                countLineData[1] = plgTmp;
                i++;
            }
            argName = `countline2.${region}=`;
            ix = vcaData[i].indexOf(argName);
            if (ix >= 0) {
                let plgTmp = vcaData[i].substring(ix + argName.length,vcaData[i].indexOf("\n"));
                countLineData[2] = plgTmp;
                i++;
            }
            argName = `countline3.${region}=`;
            ix = vcaData[i].indexOf(argName);
            if (ix >= 0) {
                let plgTmp = vcaData[i].substring(ix + argName.length,vcaData[i].indexOf("\n"));
                countLineData[3] = plgTmp;
                i++;
            }
            argName = `regionalpeoleplgx.${region}=`;
            ix = vcaData[i].indexOf(argName);
            if (ix >= 0) {
                let plgTmp = vcaData[i].substr(ix + argName.length,vcaData[i].indexOf("\n"));
                regionalPplPlgX = plgTmp;
                i++;
            }
            argName = `regionalpeoleplgy.${region}=`;
            ix = vcaData[i].indexOf(argName);
            if (ix >= 0) {
                let plgTmp = vcaData[i].substr(ix + argName.length,vcaData[i].indexOf("\n"));
                regionalPplPlgY = plgTmp;
                i++;
            }

            if (region == 4) {
                argName = "crossedline1=";
                ix = vcaData[i].indexOf(argName);
                if(ix >= 0)
                {
                    this.crossLine[0] = vcaData[i].substring(ix + argName.length,vcaData[i].indexOf("\n"));
                    i++;
                }
                argName = "crossedline2=";
                ix = vcaData[i].indexOf(argName);
                if(ix >= 0)
                {
                    this.crossLine[1] = vcaData[i].substring(ix + argName.length,vcaData[i].indexOf("\n"));
                    i++;
                }
                argName = "crossedline3=";
                ix = vcaData[i].indexOf(argName);
                if(ix >= 0)
                {
                    this.crossLine[2] = vcaData[i].substring(ix + argName.length,vcaData[i].indexOf("\n"));
                    i++;
                }
                argName = "crossedline4=";
                ix = vcaData[i].indexOf(argName);
                if(ix >= 0)
                {
                    this.crossLine[3] = vcaData[i].substring(ix + argName.length,vcaData[i].indexOf("\n"));
                    i++;
                }
                argName = "crossedline1dir=";
                ix = vcaData[i].indexOf(argName);
                if(ix >= 0)
                {
                    this.crossLineDir[0] = vcaData[i].substring(ix + argName.length,vcaData[i].indexOf("\n"));
                    i++;
                }
                argName = "crossedline2dir=";
                ix = vcaData[i].indexOf(argName);
                if(ix >= 0)
                {
                    this.crossLineDir[1] = vcaData[i].substring(ix + argName.length,vcaData[i].indexOf("\n"));
                    i++;
                }
                argName = "crossedline3dir=";
                ix = vcaData[i].indexOf(argName);
                if(ix >= 0)
                {
                    this.crossLineDir[2] = vcaData[i].substring(ix + argName.length,vcaData[i].indexOf("\n"));
                    i++;
                }
                argName = "crossedline4dir=";
                ix = vcaData[i].indexOf(argName);
                if(ix >= 0)
                {
                    this.crossLineDir[3] = vcaData[i].substring(ix + argName.length,vcaData[i].indexOf("\n"));
                    i++;
                }
            }

            if (region == 0) {
                this.regionalPeoplePlgX = regionalPplPlgX;
                this.regionalPeoplePlgY = regionalPplPlgY;
            } else {
                this.regionalPeoplePlgXPreset[region-1] = regionalPplPlgX;
                this.regionalPeoplePlgYPreset[region-1] = regionalPplPlgY;
            }
            for (i = 0; i < 4; i++) {
                if (region == 0) {
                    this.intrusionEnterPlgX[i] = enterPlgX[i];
                    this.intrusionEnterPlgY[i] = enterPlgY[i];
                    this.intrusionExitPlgX[i] = exitPlgX[i];
                    this.intrusionExitPlgY[i] = exitPlgY[i];
                    this.adMotionPlgX[i] = admotionPlgX[i];
                    this.adMotionPlgY[i] = admotionPlgY[i];
                    this.loiteringPlgX[i] = loiterPlgX[i];
                    this.loiteringPlgY[i] = loiterPlgY[i];
                    this.objectPlgX[i] = objectleftPlgX[i];
                    this.objectPlgY[i] = objectleftPlgY[i];
                    this.countLine[i] = countLineData[i];
                } else {
                    this.inrtusionEnterPresetX[region-1][i] = enterPlgX[i];
                    this.inrtusionEnterPresetY[region-1][i] = enterPlgY[i];
                    this.intrusionExitPresetX[region-1][i] = exitPlgX[i];
                    this.intrusionExitPresetY[region-1][i] = exitPlgY[i];
                    this.adMotionPresetX[region-1][i] = admotionPlgX[i];
                    this.adMotionPresetY[region-1][i] = admotionPlgY[i];
                    this.loiteringPresetX[region-1][i] = loiterPlgX[i];
                    this.loiteringPresetY[region-1][i] = loiterPlgY[i];
                    this.objectPresetX[region-1][i] = objectleftPlgX[i];
                    this.objectPresetY[region-1][i] = objectleftPlgY[i];
                    this.countLinePreset[region-1][i] = countLineData[i];
                }
            }
            if (region == 4)
                this.change_overlay(this.overlaySel);
        },
        sendOK (txt) {
            let ix = txt.indexOf("OK getalarmstatus=");
            let alarm_start = ix+18;
            let alarm_end = txt.indexOf("\n");
            // var alarm_end = txt.indexOf("OK curmaxconn=")-1;
            let code_length=alarm_end-alarm_start;
            let isSupportVca = this.sysInfo.supportvca == '1' ? true : false;
            let alarmArray = [];
            let peopleCountAlarm = false;
            if (this.isFisheye)
                isSupportVca = this.fish_support_vca(this.fishCorrect, this.fishDisplay, this.fishInstall) == 1;
            
            if (ix >= 0) {
                let code = txt.substring(code_length>8 ? (alarm_end-8): alarm_start, alarm_end);
                code = parseInt(code, 16);
                
                if (this.vcaenable == 2 && this.overlaySel == 6) {  
                    if(code & this.FLG_UI_VCA_LINECROSSING1)
                        this.show_line(1,1);
                    else
                        this.show_line(1,0);
                    if(code & this.FLG_UI_VCA_LINECROSSING2)
                        this.show_line(2,1);
                    else
                        this.show_line(2,0);
                    if(code & this.FLG_UI_VCA_LINECROSSING3)
                        this.show_line(3,1);
                    else
                        this.show_line(3,0);
                    if(code & this.FLG_UI_VCA_LINECROSSING4)
                        this.show_line(4,1);
                    else
                        this.show_line(4,0);
                }
                if(isSupportVca) {
                    let vca_ix = txt.indexOf('OK newgetalarmstatus=');
                    let tmp_ix = txt.substring(vca_ix,txt.length);
                    let vca_start = 21;
                    let vca_end = tmp_ix.indexOf('\n');
                    let r = tmp_ix.substring(vca_start,vca_end);
                    let t = r.split(';');

                    for (let i = 0; i < t.length; i++) {
                        alarmArray[i] = parseInt(t[i], 16);
                    }

                    if(this.vcaenable == 2) {
                        let alarmF = -1;
                        if (alarmArray[7] > 0) {
                            this.showCountAlarm = true;
                            peopleCountAlarm = true;
                        } else {
                            this.showCountAlarm = false;
                        }
                        switch(this.overlaySel) {
                            case 1:
                                //intrusion enter
                                if((alarmArray[1] & (1 << 0)) || (alarmArray[1] & (1 << 1)) ||
                                    (alarmArray[1] & (1 << 2)) || (alarmArray[1] & (1 << 3))) {
                                    alarmF = 1;
                                }
                                break;
                            case 2:
                                //intrusion exit
                                if((alarmArray[2] & (1 << 0)) || (alarmArray[2] & (1 << 1)) ||
                                    (alarmArray[2] & (1 << 2)) || (alarmArray[2] & (1 << 3))) {
                                    alarmF = 2;
                                }
                                break;
                            case 3:
                                //advanced motion
                                if((alarmArray[3] & (1 << 0)) || (alarmArray[3] & (1 << 1)) ||
                                    (alarmArray[3] & (1 << 2)) || (alarmArray[3] & (1 << 3))) {
                                    alarmF = 3;
                                }
                                break;
                            case 4:
                                //loitering
                                if((alarmArray[4] & (1 << 0)) || (alarmArray[4] & (1 << 1)) ||
                                    (alarmArray[4] & (1 << 2)) || (alarmArray[4] & (1 << 3))) {
                                    alarmF = 4;
                                }
                                break;
                            case 7:
                                //object_left_remove
                                let leftFlag = 0;
                                let removeFlag = 0;
                                if((alarmArray[5] & (1 << 0)) || (alarmArray[5] & (1 << 1)) ||
                                    (alarmArray[5] & (1 << 2)) || (alarmArray[5] & (1 << 3))) {
                                    leftFlag = 1;
                                }
                                if ((alarmArray[6] & (1 << 0)) || (alarmArray[6] & (1 << 1)) ||
                                    (alarmArray[6] & (1 << 2)) || (alarmArray[6] & (1 << 3))) {
                                    removeFlag = 1;
                                }
                                if (leftFlag == 1 && removeFlag == 1) {
                                    let leftCode = alarmArray[5] + ':0:-1:-1:-1:-1';
                                    let removeCode = alarmArray[6] + ':0:-1:-1:-1:-1';
                                    playerApi.MsSetRegionAlarm(leftCode);
                                    playerApi.MsSetRegionAlarm(removeCode);
                                } else {
                                    playerApi.MsSetRegionAlarm("0:0:-1:-1:-1:-1");
                                    if (leftFlag == 1) {
                                        let leftCode = alarmArray[5] + ':0:-1:-1:-1:-1';
                                        playerApi.MsSetRegionAlarm(leftCode);
                                    } 
                                    if(removeFlag == 1) {
                                        let removeCode = alarmArray[6] + ':0:-1:-1:-1:-1';
                                        playerApi.MsSetRegionAlarm(removeCode);
                                    }
                                }
                                break;
                            case 8:
                                //people count
                                if ((alarmArray[7] & (1 << 1)) || alarmArray[7] & 1) {
                                    this.show_line(5, 1);
                                    this.countLineAlarm[0] = 1;
                                } else {
                                    this.show_line(5, 0);
                                    this.countLineAlarm[0] = 0;
                                }
                                if ((alarmArray[7] & (1 << 2)) || alarmArray[7] & 1) {
                                    this.show_line(6, 1);
                                    this.countLineAlarm[1] = 1;
                                } else {
                                    this.show_line(6, 0);
                                    this.countLineAlarm[1] = 0;
                                }
                                if ((alarmArray[7] & (1 << 3)) || alarmArray[7] & 1) {
                                    this.show_line(7, 1);
                                    this.countLineAlarm[2] = 1;
                                } else {
                                    this.show_line(7, 0);
                                    this.countLineAlarm[2] = 0;
                                }
                                if ((alarmArray[7] & (1 << 4)) || alarmArray[7] & 1) {
                                    this.show_line(8, 1);
                                    this.countLineAlarm[3] = 1;
                                } else {
                                    this.show_line(8, 0);
                                    this.countLineAlarm[3] = 0;
                                }                               
                                break;
                            default:
                                break;
                        }
                        if (alarmF != -1) {
                            let vcaCode = alarmArray[alarmF] + ':0:-1:-1:-1:-1';
                            playerApi.MsSetRegionAlarm(vcaCode);
                        } else {
                            if (this.overlaySel != 10) {
                                playerApi.MsSetRegionAlarm("0:0:-1:-1:-1:-1");
                            }
                        }
                    }
                } else {
                    playerApi.MsSetRegionAlarm("0:0:-1:-1:-1:-1");
                }

                if(this.sysInfo.radarsupport == '1') {
                    if(code & this.FLG_UI_RADAR)
                        this.showRadarAlarm = true;
                    else
                        this.showRadarAlarm = false;
                }
            }    

            if (ix >= 0) {
                let code = txt.substring(code_length>8 ? (alarm_end-8): alarm_start, alarm_end);
                let code_high = code_length>8 ? txt.substring(alarm_start, alarm_end-8):'';
                code_high=parseInt(code_high, 16);
                code = parseInt(code, 16);

                let code1 = 0;
                if (code_length>8)
                    code1 = txt.substring(alarm_start, alarm_end-8);
                code1 = parseInt(code1, 16);

                if((code & 32) && !(code_high&1)) {
                    this.showRecordAlarm = true;
                } else {
                    this.showRecordAlarm = false;
                }
                if (code & 2 || code &(1 << 30) || code &(1 << 31) || code_high &(1 << 37-32) ||
                    code_high &(1 << 38-32)) {
                    this.showMotion = true;
                } else {
                    this.showMotion = false;
                }   
                if (code & 29 || code & 512 || code & this.FLG_UI_IP_CONFLICT || code & this.FLG_UI_RECORD_FAIL ||
                    code1 & this.FLG_UI_SD_FULL || code1 & this.FLG_UI_SD_UNINIT || code1 & this.FLG_UI_SD_ERROR ||
                    code1 & this.FLG_UI_NO_SD) {
                    this.showAlarm = true;
                } else if (isSupportVca) {
                    let vcaAlarmTrue = 0;
                    for (let i = 1; i <= 6; i++) {
                        if((alarmArray[i] & (1 << 0)) || (alarmArray[i] & (1 << 1)) ||
                            (alarmArray[i] & (1 << 2)) || (alarmArray[i] & (1 << 3))) {
                            vcaAlarmTrue = 1;
                            break;
                        }
                    }
                    if (code & this.FLG_UI_VCA_LINECROSSING1 || code & this.FLG_UI_VCA_CTD|| 
                        code & this.FLG_UI_VCA_LINECROSSING2 || code & this.FLG_UI_VCA_LINECROSSING3 || 
                        code & this.FLG_UI_VCA_LINECROSSING4 || 
                        code & this.FLG_UI_VCA_HUMAN) {
                        vcaAlarmTrue = 1;
                    }
                    if (vcaAlarmTrue == 1) {
                        this.showAlarm = true;
                    } else {
                        this.showAlarm = false;
                    }
                } else {
                    this.showAlarm = false;
                    this.showNewAlarm = false;
                    this.setPlayerRegionAlarm(0);
                    playerApi.MsSetRegionAlarm("0:0:-1:-1:-1:-1");
                }

                // if (isSupportVca && code & this.FLG_UI_VCA_COUNTING) {
                //     this.showCountAlarm = true;
                // } else {
                //     this.showCountAlarm = false;
                // }                 
                if (this.sysInfo.isspeed == '1' || this.sysInfo.isptzbullet == '1' || 
                    this.sysInfo.isminiptzdome == '1') {
                    this.showScheAlarm = false;
                    if (code & 1024) {
                        this.showScheAlarm = true;
                    } 
                }

                // normal liveview,has no lprLastestType
                if (this.isLprMode == 0) {
                    if (code & this.FLG_UI_LPR_BLACK)
                        this.showLprAlarm = 1;
                    else if (code & this.FLG_UI_LPR_WHITE)
                        this.showLprAlarm = 2;
                    else if (code & this.FLG_UI_LPR_VISITOR)
                        this.showLprAlarm = 3;
                    else
                        this.showLprAlarm = 0;
                } else {
                    if ((code & this.FLG_UI_LPR_BLACK) && this.lprLastestType == 1)
                        this.showLprAlarm = 1;
                    else if ((code & this.FLG_UI_LPR_WHITE) && this.lprLastestType == 2)
                        this.showLprAlarm = 2;
                    else if ((code & this.FLG_UI_LPR_VISITOR) && this.lprLastestType == 3)
                        this.showLprAlarm = 3;
                    else
                        this.showLprAlarm = 0;
                }             

                ix = txt.indexOf("OK curmaxconn=");
                if (ix >= 0) {
                    //The number of connections may be greater than nine
                    let Num = txt.substring(ix + 14, txt.indexOf(";"));
                    playerApi.MsSetParam("currentConnects", Num);
                    this.curConn = Num;
                }

                let smartStramArr = ['', '', '', '', ''];
                ix = txt.indexOf('OK smartstreamswitch=');
                if (ix > 0)
                    smartStramArr[0] = txt.charAt(ix + 21);
                ix = txt.indexOf('OK subsmartstream=');
                if (ix > 0)
                    smartStramArr[1] = txt.charAt(ix + 18);
                ix = txt.indexOf('OK thirdsmartstream=');
                if (ix > 0)
                    smartStramArr[2] = txt.charAt(ix + 20);
                ix = txt.indexOf('OK fourthsmartstream=');
                if (ix > 0)
                    smartStramArr[3] = txt.charAt(ix + 21);
                ix = txt.indexOf('OK fifthsmartstream=');
                if (ix > 0)
                    smartStramArr[4] = txt.charAt(ix + 20);

                if (this.isFisheye) {
                    for (let i=0; i<5; i++) {
                        if (this.curWndPlay[i] == 1)
                            playerApi.MsSetParam(`smartstreamon${i}`, smartStramArr[this.playUrlIndex[i]]);
                    }
                } else {
                    playerApi.MsSetParam('smartstreamon', smartStramArr[this.streamSel]);
                }

                if (this.isFisheye) {
                    let flag = 0;
                    ix = txt.indexOf('OK fishcorrectmodel=');
                    if (ix >= 0) {
                        if (txt.charAt(ix + 20) != this.fishCorrect)
                            flag = 1;
                    }
                    ix = txt.indexOf('OK fishinstallmodel=');
                    if (ix >= 0) {
                        if (txt.charAt(ix + 20) != this.fishInstall)
                            flag = 1;
                    }
                    ix = txt.indexOf('OK fishdisplaymodel=');
                    if (ix >= 0) { 
                        if (txt.charAt(ix + 20) != this.fishDisplay)
                            flag = 1;
                    }
                    if (flag == 1) {
                        ASTGUI.cookies.setCookie("pageName","livevideo.html");
                        // window.location.href = "index.html";
                        window.location.reload();
                    }
                }

                // 翻转之后，修改vca lpr检测区域 TODO
                if (this.vcaenable == 2 || this.sysInfo.supportlpr == '1') {
                    let flag = 0;
                    ix = txt.indexOf("OK corridormode=");
                    if(ix >= 0) {
                        var corridormode = txt.charAt(ix+16);
                        if(this.curcorridormode != corridormode) {
                            this.curcorridormode = corridormode;
                            flag = 1;
                        }
                    }
                    ix = txt.indexOf("OK imagerotation=");
                    if(ix >= 0) {
                        var imagerotation = txt.charAt(ix+17);
                        if(this.curimagerotation != imagerotation) {
                            this.curimagerotation = imagerotation;
                            flag = 1;
                        }
                    }
                    ix = txt.indexOf("OK curposinpreset=");
                    if (ix >= 0) {
                        let tmp = txt.substring(ix + 18);
                        let curpreset = tmp.substring(0, tmp.indexOf("\n"));
                        curpreset = parseInt(curpreset);
                        if(this.curPresetPos != curpreset) {
                            this.notify_update_preset(curpreset);
                            this.curPresetPos = curpreset;
                            this.vcaCurPreset = curpreset;
                            flag = 1;
                        }
                    }
                    if(flag == 1) {
                        let k = 0;
                        for (k = 0; k <= 4; k++) {
                            let sUrl = `paratest=intrusionenterplgx.${k}`;
                            sUrl += `&paratest=intrusionenterplgy.${k}`;
                            sUrl += `&paratest=intrusionexitplgx.${k}`;
                            sUrl += `&paratest=intrusionexitplgy.${k}`;
                            sUrl += `&paratest=admotionplgx.${k}`;
                            sUrl += `&paratest=admotionplgy.${k}`;
                            sUrl += `&paratest=loiteringplgx.${k}`;
                            sUrl += `&paratest=loiteringplgy.${k}`;
                            sUrl += `&paratest=objectplgx.${k}`;
                            sUrl += `&paratest=objectplgy.${k}`;
                            sUrl += `&paratest=countline.${k}`;
                            sUrl += `&paratest=countline1.${k}`;
                            sUrl += `&paratest=countline2.${k}`;
                            sUrl += `&paratest=countline3.${k}`;
                            sUrl += `&paratest=regionalpeoleplgx.${k}`;
                            sUrl += `&paratest=regionalpeoleplgy.${k}`;
                            if ( k == 4) {
                                sUrl += "&paratest=crossedline1&paratest=crossedline2&paratest=crossedline3&paratest=crossedline4";
                                sUrl += "&paratest=crossedline1dir&paratest=crossedline2dir&paratest=crossedline3dir&paratest=crossedline4dir";
                            }
                            send_vb_htm(sUrl).then( res => {
                                this.update_vca_mask(res)
                            });
                        }
                    }
                }

                ix = txt.indexOf("OK getiotalarmstatus=");
                if (ix >= 0) {
                    let code = txt.substring(ix + 21, ix + 29);
                    code = parseInt(code, 16);
                    if(code == 0)
                        this.showIotAlarm = false;
                    else
                        this.showIotAlarm = true;
                }

                ix = txt.indexOf("OK regionalpeoplealarmstatus=");
                if (ix >= 0) {
                    let alarm_start = ix+29;
                    let code = txt.substring(alarm_start);					
                    code = parseInt(code, 16);
                    if (!peopleCountAlarm) {
                        if(code > 0) {
                            this.showCountAlarm = true;
                        } else {
                            this.showCountAlarm = false;
                        }
                    }
                }
            }
            if (this.hasDestroy) {  // clear before quit
                return;
            }
            if (this.alarmTimerFlag) {
                this.alarmTimer = setTimeout( () => {
                    this.StartAlarmWorker();
                }, 1000);
            }
        },
        StartAlarmWorker () {
            let url = 'language=ie&getalarmstatus&curmaxconn&newgetalarmstatus=-1';
            if (this.sysInfo.isfisheye == '1') {
                this.getFishTime++;
                if (this.getFishTime > 5) {
                    url += "&paratest=fishcorrectmodel&paratest=fishdisplaymodel&paratest=fishinstallmodel";
                    this.getFishTime = 0;
                }
            }
            if (this.vcaenable == 2 || this.sysInfo.supportlpr == '1') {
                this.getPresetTime++;
                if (this.getPresetTime >= 5) {
                    url += "&paratest=curposinpreset&paratest=corridormode&paratest=imagerotation";
                    this.getPresetTime = 0;
                }
            }

            if (this.iotSupport != 0 ) {
                url += "&getiotalarmstatus";
            }
            if ((this.regionalPeopleEnable[0] || this.regionalPeopleEnable[1] || this.regionalPeopleEnable[2] || this.regionalPeopleEnable[3]) &&
                (this.sysInfo.isnewhi3516dv300 == '1' || this.sysInfo.iscv2xplatform == '1'))
            {
                url += "&regionalpeoplealarmstatus";
            }
            if (this.sysInfo.ishisi3516 == '0') {
                url += "&paratest=fourthsmartstream&paratest=fifthsmartstream";
            }

            url += "&paratest=smartstreamswitch&paratest=subsmartstream&paratest=thirdsmartstream";
            send_vb_htm(url).then( res => {
                this.sendOK(res)
            });
        },
        triggleRight(){
            this.showRight = !this.showRight
        },
        change_face_lays (val) {
            this.faceSel = val;
            ASTGUI.cookies.setCookie("faceType", val);
            if (val == 0) {
                this.closeFaceDetection();
            } else {
                this.showFaceDetection();
            }
        },
        showFaceDetection () {
            playerApi.MsSetPolygonMap(1, 1, this.facedetectplgx, this.facedetectplgy);
        },
        closeFaceDetection () {
            playerApi.MsSetPolygonMap(1, 1, '-1:-1:-1:-1:-1:-1:-1:-1:-1:-1', '-1:-1:-1:-1:-1:-1:-1:-1:-1:-1');
        },
        changeFaceMode (first) {
            this.isFaceMode = !this.isFaceMode;
            this.visible = false;   // close manual output,because layout will be changed
            if (this.isFaceMode) {
                let vcaType = ASTGUI.cookies.getCookie("vcaType");
                this.change_overlay(0);		//close vca draw
                ASTGUI.cookies.setCookie("vcaType",vcaType);
                this.showOverlay = false;
                if (first == '1') {
                    this.faceLayTimer = setTimeout(() => {
                        this.change_face_lays(this.faceSel);
                    }, 2000);
                } else {
                    this.change_face_lays(this.faceSel);
                }       
                this.enableFace();
            } else {
                playerApi.MsSetFaceRectRegion(0, "0:0:0:0:0:0");
                clearTimeout(this.facePointTimer);
                clearTimeout(this.facePicTimer);
                this.closeFaceDetection();
                this.showOverlay = true;
                let vcaType = ASTGUI.cookies.getCookie("vcaType");
                this.overlaySel = IsNull(vcaType)? 0 : parseInt(vcaType);
                this.change_overlay(this.overlaySel);
            }
            let mode = this.isFaceMode ? "1" : "0";
            ASTGUI.cookies.setCookie("facemode", mode);
        },
        initFaceShow() {
            let faceMode = (IsNull(ASTGUI.cookies.getCookie("facemode"))?0:parseInt(ASTGUI.cookies.getCookie("facemode")));
            this.isFaceMode = !faceMode;
            let faceType = ASTGUI.cookies.getCookie("faceType");
            this.faceSel = IsNull(faceType)? 0 : parseInt(faceType);
            this.changeFaceMode('1');
        },
        enableFace () {
            clearTimeout(this.facePointTimer);
            clearTimeout(this.facePicTimer);
            if (this.isFaceMode) {
                this.updateFace();
                this.updateFacePic();
            }
        },
        updateFace () {
            let url = 'facepoint';
            send_vb_htm(url).then( res => {
                let txt = res;
                let ix = txt.indexOf('OK facepoint=');
                if (ix >= 0) {
                    let code = txt.substring(ix + 12);
                    if (!this.isFaceMode) {
                        playerApi.MsSetFaceRectRegion(0, "0:0:0:0:0:0");
                    } else {
                        playerApi.MsSetFaceRectRegion(1, code);
                    }
                }
                if (this.isFaceMode) {
                    this.facePointTimer = setTimeout(() => {
                        this.updateFace();
                    }, 100);
                }
            });
        },
        updateFacePic () {
            let url = 'getfacelocation';
            send_vb_htm(url).then( res => {
                let txt = res;
                let ix = txt.indexOf('OK getfacelocation=');
                if (ix >= 0) {
                    let code = txt.substring(ix + 19);
                    let faceData = code.split("&");

                    if (faceData.length > 0) {
                        let j = faceData.length - 1;
                        for (let i = 0; i < j; i++) {
                            let picName = "";
                            let picTime = "";
                            let genderPic = "icon-unrecognized";
                            let agePic = "icon-unrecognized";
                            let glassesPic = "icon-unrecognized";
                            let maskPic = "icon-unrecognized";
                            let genderD = "-";
                            let ageD = "-";
                            let glassesD = "-";
                            let maskD = "-";
                            
                            let jpgData = faceData[j - i - 1].split(";");
                            this.faceAttibuteData[i] = jpgData[0].substring(0, 4)+"-"+jpgData[0].substring(4, 6)+
                                "-"+jpgData[0].substring(6, 8);
                            this.faceAttibuteData[i] += ";" + jpgData[0].substring(8, 10)+":"+
                                jpgData[0].substring(10, 12)+":"+ jpgData[0].substring(12, 14);
                            this.faceAttibuteData[i] += ";" + jpgData[0] + "_" + jpgData[1] + "_" +
                                 jpgData[2] + "_" + jpgData[3];
                            this.faceAttibuteData[i] += ";" + jpgData[4] + ";" + jpgData[5] + ";" + jpgData[6] +
                                 ";" + jpgData[7] + ";" + jpgData[8] + ";" + jpgData[9];
                            if (jpgData[10] == "0") {
                                this.faceAttibuteData[i] += ";0"
                            } else {
                                this.faceAttibuteData[i] += ";" + jpgData[10];
                            }

                            picName = jpgData[0] + "_" + jpgData[1] + "_" + jpgData[2] + "_" + jpgData[3];
                            picTime = jpgData[0].substring(8, 10)+":"+jpgData[0].substring(10, 12)+
                                ":"+jpgData[0].substring(12, 14);

                            if (jpgData[5] == "0") {
                                genderPic = "icon-male";
                                genderD	  = this.$t('male');
                            } else if (jpgData[5] == "1") {
                                genderPic = "icon-female";
                                genderD	  = this.$t('female');
                            }
                            
                            if (parseInt(jpgData[4]) >= 0 && parseInt(jpgData[4]) <= 17) {
                                agePic = "icon-child";
                                ageD   = this.$t('child');
                            } else if (parseInt(jpgData[4]) >= 18 && parseInt(jpgData[4]) <= 59) {
                                agePic = "icon-adult";
                                ageD   = this.$t('adult');
                            } else if (parseInt(jpgData[4]) > 59) {
                                agePic = "icon-elderly";
                                ageD   = this.$t('elderly');
                            }

                            if (parseInt(jpgData[6]) == 0) {
                                glassesPic = "icon-no-glasses1";
                                glassesD = this.$t('no');
                            } else if (parseInt(jpgData[6]) == 1) {
                                glassesPic = "icon-glasses";
                                glassesD = this.$t('yes');
                            }

                            if (jpgData[7] == "0") {
                                maskPic = "icon-no-mask";
                                maskD = this.$t('no');
                            } else if (jpgData[7] == "1") {
                                maskPic = "icon-mask";
                                maskD = this.$t('yes');
                            }

                            let pic = '/face/' + picName + '.jpg';
                            this.faceTable[i].src = pic;
                            this.faceTable[i].genderClass = genderPic;
                            this.faceTable[i].genderTitle = genderD;
                            this.faceTable[i].ageClass = agePic;
                            this.faceTable[i].ageTitle = ageD;
                            this.faceTable[i].glassClass = glassesPic;
                            this.faceTable[i].glassTitle = glassesD;
                            this.faceTable[i].maskClass = maskPic;
                            this.faceTable[i].maskTitle = maskD;
                            this.faceTable[i].time = picTime;
                            this.faceTable[i].show = true;
                        }
                        for (var i = j; i < 20; i++) {
                            this.faceTable[i].show = false;
                        }
                    }
                }
                if (this.isFaceMode) {
                    this.facePointTimer = setTimeout(() => {
                        this.updateFacePic();
                    }, 3000);
                }
            });
        },
        faceAttribute (index) {
            this.curGender = '-';
            this.curAge = '-';
            this.curGlasses = '-';
            this.curMask = '-';
            this.curCap = '-';
            let attrData = this.faceAttibuteData[index].split(";");
            this.curCaptureTime = attrData[0] + " " + attrData[1];
            if (parseInt(attrData[3]) >= 0 && parseInt(attrData[3]) <= 17) {
                this.curAge = this.$t('child');
            } else if (parseInt(attrData[3]) >= 18 && parseInt(attrData[3]) <= 59) {
                this.curAge = this.$t('adult');
            } else if (parseInt(attrData[3]) > 59) {
                this.curAge = this.$t('elderly');
            }
            if (attrData[4] == "0") {
                this.curGender = this.$t('male');
            } else if (attrData[4] == "1") {
                this.curGender = this.$t('female');
            }
            if (attrData[5] == "0") {
                this.curGlasses = this.$t('no');
            } else if (attrData[5] == "1") {
                this.curGlasses = this.$t('yes');
            }
            if (attrData[6] == "0") {
                this.curMask = this.$t('no');
            } else if (attrData[6] == "1") {
                this.curMask = this.$t('yes');
            }
            if (attrData[7] == "0") {
                this.curCap = this.$t('no');
            } else if (attrData[7] == "1") {
                this.curCap = this.$t('yes');
            }
            let url = "url(/face/"+attrData[2]+".jpg)";
            this.facePicUrl = url;
            if (attrData[9] == '0') {
                this.faceBkgUrl = '';
            } else {
                this.faceBkgUrl = "url(/bkgface/"+attrData[9]+")";
            }
            this.showFaceDetail = true;
        },
        send_vca_install (val) {
            if (this.vcaInstall == val)
                return ;
            if (this.isPlaying > 0) { 
                this.click_play() //先停止播放
            }
            this.vcaInstall = val;
            set_page_data('vcasettings', `&cameraview=${val}`);
            this.loading = show_loading();
			setTimeout(() => {
				get_reload_flag(() => {
					this.loading.close();
					location.reload();
				});
			}, 30000);
        },
        async get_config_info () {
            let param = '&zoompos&irismode&iristype&brightness&contrast&saturation&sharpness&nflevel&dnr2level&autoscan' +
                '&lightstatus&pos3denable&onepatrol&watchenable&trackenable&defogstatus&recordtime' +
                '&smartstreamswitch&subsmartstream&thirdsmartstream&fishinstallmodel&fishdisplaymodel&fishcorrectmodel' +
                '&intrusionenterplgx.0-4&intrusionenterplgy.0-4&intrusionexitplgx.0-4&intrusionexitplgy.0-4' +
                '&admotionplgx.0-4&admotionplgy.0-4+&loiteringplgx.0-4&loiteringplgy.0-4&objectplgx.0-4&objectplgy.0-4' +
                '&intrusionenterenable.0-3&intrusionexitenable.0-3&admotionenable.0-3 + &loiteringenable.0-3' +
                '&counterenable.0-4&personenable&objectleftenable.0-3&lineenable.0-3&crossedline1.0-4&crossedline2.0-4' +
                '&crossedline3.0-4&crossedline4.0-4&crossedline1dir.0-4&crossedline2dir.0-4&crossedline3dir.0-4&crossedline4dir.0-4'+
                '&countline.0-4&showautotrack&regionalenable.0-3&regionalpeoleplgx.0-4&regionalpeoleplgy.0-4&pristreamw&pristreamh' +
                '&substreamw&substreamh&tristreamw&tristreamh&enablesubstream&enablethirdstream&enablefourthstream' +
                '&enablefifthstream&outputduration&outputdurationex&patrolrecoveryenable&patrolrecoverytime' +
                '&trackmanualenable&watchtime&watchmode&watchid&lprscene&lprroi.0-3&lprroi1.0-3&lprroi2.0-3&lprroi3.0-3' +
                '&lprroi4.0-3&curposinpreset&lprreswidth&lprresheight&lprlicensestatus&lprcountry&lprsnapseparator' +
                '&lprsnapsort&lprsnapposition&lprsnapid&title&presetid&presetnameencode&tourset.0-7&tourplaystatus.0-7' +
                '&patternrecord.0-3&patternstatus.0-3&patternplaystatus.0-3&ptzspeed&outputactionTime&outputactionTimeex' +
                '&caniris&isnewbox&vcaenable&facedetectplgx&facedetectplgy&audioenable&audiomode&issupportspeaker' +
                `&countline1.0-4&countline2.0-4&countline3.0-4` +
                `&countlinedir.0-4&countline1dir.0-4&countline2dir.0-4&countline3dir.0-4&httpsport` +
                '&fish1Osubenable.1&fish1Psubenable.3' +
                '&fish2Psubenable.5&fish4Rsubenable.7&fish1O3Rsubenable.11&fish1P3Rsubenable.15&fishpresetid.0-4' +
                '&fishpresetnameencode.0-4&fishtourset0.0-7&fishtourset1.0-7&fishtourset2.0-7&fishtourset3.0-7' +
                '&fishtourset4.0-7&audioenable&maxconn&smarteventenable&ptzmotionenable.0&lprenable&isfisheye' +
                '&patrolrecoveryenable&fishautoscan.0-4&facemscenable&cameraview&issupportliveviewlen&radarenable' +
                '&osdzmtime&osdmdtime&osdpatroltime&osdscantime&outputstatus&outputstatusex&corridormode&imagerotation'+
                '&regionentrancescene&regionexitscene&advancedmotionscene&loiteringscene&leftremovescene&linecrossingscene'+
                '&regionalpeoplescene&countlinescene&supportIoT&curmaxconn&fishptzposition&mirctrl';

            let res = await get_config_data(param);
                let info = res.config_par[0];
                if (is_support_audio_settings() && info.audioenable == '1') {
                    if (info.audiomode != '1') {
                        this.showMute = true;
                    }
                    if (info.audiomode != '0' && info.issupportspeaker == '1') {
                        this.showSpeaker = true;
                    }
                }
                this.fishPtzRegion = info.fishptzposition
                this.curConn = parseInt(info.curmaxconn);
                this.entrancescene = parseInt(info.regionentrancescene) ? 1 : 0;
                this.exitscene = parseInt(info.regionexitscene) ? 1 : 0;
                this.motionscene = parseInt(info.advancedmotionscene) ? 1 : 0;
                this.loiteringscene = parseInt(info.loiteringscene) ? 1 : 0;
                this.leftremovescene = parseInt(info.leftremovescene) ? 1 : 0;
                this.linecrossingscene = parseInt(info.linecrossingscene) ? 1 : 0;
                this.regionalpeoplescene = parseInt(info.regionalpeoplescene) ? 1 : 0;
                this.countlinescene = parseInt(info.countlinescene) ? 1 : 0;
                this.alarmType = parseInt(info.outputactionTime);
                this.alarmTypeEx = parseInt(info.outputactionTimeex);
                this.alarmDuration = parseInt(info.outputduration);
                this.alarmDurationEx = parseInt(info.outputdurationex);
                this.isStart = parseInt(info.outputstatus);
                this.iotSupport = parseInt(info.supportIoT);
            	this.lightMode = parseInt(info.lightstatus);
                if (this.alarmType != -2) {
                    this.isStart = false;
                }
                this.isStart1 = parseInt(info.outputstatusex);
                if (this.alarmTypeEx != -2) {
                    this.isStart1 = false;
                }
                this.magnifier = parseInt(info.zoompos);
                this.zoomPos = parseInt(info.zoompos);
                this.bright = parseInt(parseInt(info.brightness) * 100 / 255);
                this.contrast = parseInt(parseInt(info.contrast) * 100 / 255);
                this.saturation = parseInt(parseInt(info.saturation) * 100 / 255);
                this.sharp = parseInt(parseInt(info.sharpness) * 100 / 255);
                this.dnr2d = parseInt(parseInt(info.dnr2level) * 100 / 255);
                this.nflevel = parseInt(parseInt(info.nflevel) * 100 / 255);
                this.irisMode = parseInt(info.irismode);
                this.irisType = parseInt(info.iristype);
                if (info.caniris == '0' && !(info.isnewbox == '1' && (info.iristype == '1' || info.iristype == '2'))) {
                    this.noManualIris = true;
                    this.noAutoIris = true;
                }
                this.caniris = parseInt(info.caniris);
                this.mainSmartStream = parseInt(info.smartstreamswitch);
                this.subSmartStream = parseInt(info.subsmartstream);
                this.thirdSmartStream = parseInt(info.thirdsmartstream);
                this.curDisplay = this.fishDisplay = parseInt(info.fishdisplaymodel);
                this.curFishInstall = this.fishInstall = parseInt(info.fishinstallmodel);
                this.fishCorrect = parseInt(info.fishcorrectmodel);
                if (info.isfisheye == '1') {
                    this.$store.dispatch("fisheye/setFishDisplay", this.curDisplay);
                    this.$store.dispatch("fisheye/setFishInstall", this.curFishInstall);
                    this.$store.dispatch("fisheye/setFishCorrect", this.fishCorrect);
                }
                this.recordTime = parseInt(info.recordtime);
                let enterPlgX = info['intrusionenterplgx.0'].split(';');
                let enterPlgY = info['intrusionenterplgy.0'].split(';');
                let exitPlgX = info['intrusionexitplgx.0'].split(';');
                let exitPlgY = info['intrusionexitplgy.0'].split(';');
                let admotionPlgX = info['admotionplgx.0'].split(';');
                let admotionPlgY = info['admotionplgy.0'].split(';');

                let loiterPlgX = info['loiteringplgx.0'].split(';');
                let loiterPlgY = info['loiteringplgy.0'].split(';');
                let objectleftPlgX = info['objectplgx.0'].split(';');
                let objectleftPlgY = info['objectplgy.0'].split(';');

                let enterPresetX, enterPresetY, exitPresetX, exitPresetY;
                let admotionPresetX, admotionPresetY, loiterPresetX, loiterPresetY;
                let objectleftPresetX, objectleftPresetY;
                let i = 0, j = 0;
                for(i=0; i<4; i++) {
                    this.intrusionEnterPlgX[i] = enterPlgX[i];
                    this.intrusionEnterPlgY[i] = enterPlgY[i];
                    this.intrusionExitPlgX[i] = exitPlgX[i];
                    this.intrusionExitPlgY[i] = exitPlgY[i];
                    this.adMotionPlgX[i] = admotionPlgX[i];
                    this.adMotionPlgY[i] = admotionPlgY[i];
                    this.loiteringPlgX[i] = loiterPlgX[i];
                    this.loiteringPlgY[i] = loiterPlgY[i];
                    this.objectPlgX[i] = objectleftPlgX[i];
                    this.objectPlgY[i] = objectleftPlgY[i];
                    enterPresetX = info[`intrusionenterplgx.${i+1}`].split(';');
                    enterPresetY = info[`intrusionenterplgy.${i+1}`].split(';');
                    exitPresetX = info[`intrusionexitplgx.${i+1}`].split(';');
                    exitPresetY = info[`intrusionexitplgy.${i+1}`].split(';');
                    admotionPresetX = info[`admotionplgx.${i+1}`].split(';');
                    admotionPresetY = info[`admotionplgy.${i+1}`].split(';');
                    loiterPresetX = info[`loiteringplgx.${i+1}`].split(';');
                    loiterPresetY = info[`loiteringplgy.${i+1}`].split(';');
                    objectleftPresetX = info[`objectplgx.${i+1}`].split(';');
                    objectleftPresetY = info[`objectplgy.${i+1}`].split(';');
                    for (j = 0; j < 4; j++) {
                        this.inrtusionEnterPresetX[i][j] = enterPresetX[j];
                        this.inrtusionEnterPresetY[i][j] = enterPresetY[j];
                        this.intrusionExitPresetX[i][j] = exitPresetX[j];
                        this.intrusionExitPresetY[i][j] = exitPresetY[j];
                        this.adMotionPresetX[i][j] = admotionPresetX[j];
                        this.adMotionPresetY[i][j] = admotionPresetY[j];
                        this.loiteringPresetX[i][j] = loiterPresetX[j];
                        this.loiteringPresetY[i][j] = loiterPresetY[j];
                        this.objectPresetX[i][j] = objectleftPresetX[j];
                        this.objectPresetY[i][j] = objectleftPresetY[j];
                    }
                }
                this.intrusionEnterEnable = [info['intrusionenterenable.0'] == '1'? true : false,
                    info['intrusionenterenable.1'] == '1'? true : false,
                    info['intrusionenterenable.2'] == '1'? true : false,
                    info['intrusionenterenable.3'] == '1'? true : false];
                this.intrusionExitEnable = [info['intrusionexitenable.0'] == '1'? true : false,
                    info['intrusionexitenable.1'] == '1'? true : false,
                    info['intrusionexitenable.2'] == '1'? true : false,
                    info['intrusionexitenable.3'] == '1'? true : false];
                this.adMotionEnable = [info['admotionenable.0'] == '1'? true : false,
                    info['admotionenable.1'] == '1'? true : false,
                    info['admotionenable.2'] == '1'? true : false,
                    info['admotionenable.3'] == '1'? true : false];
                this.loiteringEnable = [info['loiteringenable.0'] == '1'? true : false,
                    info['loiteringenable.1'] == '1'? true : false,
                    info['loiteringenable.2'] == '1'? true : false,
                    info['loiteringenable.3'] == '1'? true : false];
                this.objectEnable = [info['objectleftenable.0'] == '1'? true : false,
                    info['objectleftenable.1'] == '1'? true : false,
                    info['objectleftenable.2'] == '1'? true : false,
                    info['objectleftenable.3'] == '1'? true : false];         
                this.lineEnable = [parseInt(info['lineenable.0']), parseInt(info['lineenable.1']), parseInt(info['lineenable.2']), parseInt(info['lineenable.3'])];
                this.crossLine = [info['crossedline1.0'], info['crossedline2.0'], info['crossedline3.0'], info['crossedline4.0']];
                this.crossLineDir = [parseInt(info['crossedline1dir.0']), parseInt(info['crossedline2dir.0']),
                                    parseInt(info['crossedline3dir.0']), parseInt(info['crossedline4dir.0'])];
                this.crossLinePreset = [[info['crossedline1.1'], info['crossedline2.1'], info['crossedline3.1'], info['crossedline4.1']],
                                        [info['crossedline1.2'], info['crossedline2.2'], info['crossedline3.2'], info['crossedline4.2']],
                                        [info['crossedline1.3'], info['crossedline2.3'], info['crossedline3.3'], info['crossedline4.3']],
                                        [info['crossedline1.4'], info['crossedline2.4'], info['crossedline3.4'], info['crossedline4.4']]];
                this.crossLineDirPreset = [[parseInt(info['crossedline1dir.1']), parseInt(info['crossedline2dir.1']),
                                            parseInt(info['crossedline3dir.1']), parseInt(info['crossedline4dir.1'])],
                                        [parseInt(info['crossedline1dir.2']), parseInt(info['crossedline2dir.2']),
                                            parseInt(info['crossedline3dir.2']), parseInt(info['crossedline4dir.2'])],
                                        [parseInt(info['crossedline1dir.3']), parseInt(info['crossedline2dir.3']),
                                            parseInt(info['crossedline3dir.3']), parseInt(info['crossedline4dir.3'])],
                                        [parseInt(info['crossedline1dir.4']), parseInt(info['crossedline2dir.4']),
                                            parseInt(info['crossedline3dir.4']), parseInt(info['crossedline4dir.4'])]];
                this.countEnable = [parseInt(info['counterenable.0']), parseInt(info['counterenable.1']), 
                                    parseInt(info['counterenable.2']), parseInt(info['counterenable.3'])];
                this.countLine = [info['countline.0'], info['countline1.0'], info['countline2.0'], info['countline3.0']];
                this.countLineDir = [parseInt(info['countlinedir.0']), parseInt(info['countline1dir.0']),
                                    parseInt(info['countline2dir.0']), parseInt(info['countline3dir.0'])];
                this.countLinePreset = [[info['countline.1'], info['countline1.1'], info['countline2.1'], info['countline3.1']],
                                        [info['countline.2'], info['countline1.2'], info['countline2.2'], info['countline3.2']],
                                        [info['countline.3'], info['countline1.3'], info['countline2.3'], info['countline3.3']],
                                        [info['countline.4'], info['countline1.4'], info['countline2.4'], info['countline3.4']]];
                this.countLineDirPreset = [[parseInt(info['countlinedir.1']), parseInt(info['countline1dir.1']), parseInt(info['countline2dir.1']), parseInt(info['countline3dir.1'])],
                                        [parseInt(info['countlinedir.2']), parseInt(info['countline1dir.2']), parseInt(info['countline2dir.2']), parseInt(info['countline3dir.2'])],
                                        [parseInt(info['countlinedir.3']), parseInt(info['countline1dir.3']), parseInt(info['countline2dir.3']), parseInt(info['countline3dir.3'])],
                                        [parseInt(info['countlinedir.4']), parseInt(info['countline1dir.4']), parseInt(info['countline2dir.4']), parseInt(info['countline3dir.4'])]];
                this.trackEnable = parseInt(info.trackenable);
                this.showTrack = parseInt(info.showautotrack);
                this.manualTrack = parseInt(info.trackmanualenable);
                set_manual_track(this.manualTrack);
                this.regionalPeopleEnable = [parseInt(info['regionalenable.0']), parseInt(info['regionalenable.1']), 
                    parseInt(info['regionalenable.2']), parseInt(info['regionalenable.3'])];
                this.regionalPeoplePlgX = info['regionalpeoleplgx.0'];
                this.regionalPeoplePlgY = info['regionalpeoleplgy.0'];
                this.regionalPeoplePlgXPreset = [info['regionalpeoleplgx.1'], info['regionalpeoleplgx.2'],
                                                info['regionalpeoleplgx.3'], info['regionalpeoleplgx.4']];
                this.regionalPeoplePlgYPreset = [info['regionalpeoleplgy.1'], info['regionalpeoleplgy.2'],
                                                info['regionalpeoleplgy.3'], info['regionalpeoleplgy.4']];
                this.mainResW = parseInt(info.pristreamw);
                this.mainResH = parseInt(info.pristreamh);
                this.subResW = parseInt(info.substreamw);
                this.subResH = parseInt(info.substreamh);
                this.triResW = parseInt(info.tristreamw);
                this.triResH = parseInt(info.tristreamh);
                this.subStreamEnable = parseInt(info.enablesubstream);
                this.triStreamEnable = parseInt(info.enablethirdstream);
                this.fourStreamEnable = parseInt(info.enablefourthstream);
                this.fifStreamEnable = parseInt(info.enablefifthstream);
                this.httpsPort = parseInt(info.httpsport);
                this.watchEnable = parseInt(info.watchenable);
                this.watchTime = parseInt(info.watchtime);
                this.watchMode = parseInt(info.watchmode);
                this.watchId = parseInt(info.watchid);
                this.manualDefog = parseInt(info.defogstatus);
                if (this.manualDefog) {
                    this.defogTimer = setTimeout( () => {
                        this.update_defog_status();
                    }, 1000);
                }
                this.facedetectplgx = info.facedetectplgx;
                this.facedetectplgy = info.facedetectplgy;
                this.lprScene = parseInt(info.lprscene);
                this.lprRoiRegion = [info['lprroi.0'], info['lprroi.1'], info['lprroi.2'], info['lprroi.3']];
                for (let i=0; i<this.LPR_PRESET_NUM; i++) {
                    let t = [info[`lprroi${i + 1}.0`], info[`lprroi${i + 1}.1`], info[`lprroi${i + 1}.2`],
                        info[`lprroi${i + 1}.3`]];
                    this.lprRoiRegionPreset.push(t);
                }
                this.curPresetPos = parseInt(info.curposinpreset);
                this.vcaCurPreset = parseInt(info.curposinpreset);
                this.curcorridormode = parseInt(info.corridormode);
                this.curimagerotation = parseInt(info.imagerotation);
                this.lprWidth = parseInt(info.lprreswidth);
                this.lprHeight = parseInt(info.lprresheight);
                this.lprStatus = parseInt(info.lprlicensestatus);
                if (info.lprcountry == '19')
                    this.isIran = true;
                this.lprSeparator = info.lprsnapseparator;
                this.lprSnapSort = info.lprsnapsort;
                this.lprSnapPos = info.lprsnapposition;
                this.lprSnapId = info.lprsnapid;
                this.deviceName = info.title;
                this.autoScanStatus = parseInt(info.autoscan) ? true : false;
                this.ptzSpeed = parseInt(info.ptzspeed) + 1;
                this.posEnable = parseInt(info.pos3denable);
                this.enableAudio = parseInt(info.audioenable) ? true : false;
                this.showLensDiv = parseInt(info.issupportliveviewlen) ? true : false;
                this.maxConn = parseInt(info.maxconn);
                this.smartEventEnable = parseInt(info.smarteventenable);
                this.ptzMotionEnable = parseInt(info['ptzmotionenable.0']);
                this.lprEnable = parseInt(info.lprenable);
                this.patrolRecoveryEnable = parseInt(info.patrolrecoveryenable);
                this.patrolMode = parseInt(info.onepatrol);
                this.vcaInstall = parseInt(info.cameraview);
                if (info.presetid != '') {
                    this.presetArr = info.presetid.split(',');
                    for (let i=0; i<this.presetArr.length; i++) {
                        this.presetArr[i] = parseInt(this.presetArr[i]);
                    }
                    this.presetName = info.presetnameencode.split(',');
                    for (let i=0; i<this.presetName.length; i++) {
                        this.presetName[i] = ASTGUI.decode(this.presetName[i]);
                    }
                }
                for (let i=0; i<this.FISH_MAX_VIEW_NUM; i++) {
                    let str = info[`fishpresetid.${i}`];
                    if (str) {
                        this.fishPresetArr[i] = str.split(',');
                        for (let j=0; j<this.fishPresetArr[i].length; j++) {
                            this.fishPresetArr[i][j] = parseInt(this.fishPresetArr[i][j]);
                        }
                        this.fishPresetName[i] = info[`fishpresetnameencode.${i}`].split(',');
                        for (let j=0; j<this.fishPresetName[i].length; j++) {
                            this.fishPresetName[i][j] = ASTGUI.decode(this.fishPresetName[i][j]);
                        }
                    }
                }
                for (let i=0; i<this.PATROL_NUM; i++) {
                    if (info[`tourset.${i}`]) {
                        let arr = info[`tourset.${i}`].split(';');
                        if (arr == '')
                            continue ;
                        let pid = arr[0].split(':');
                        let speed = arr[1].split(':');
                        let time = arr[2].split(':');
                        for (let j=0; j<pid.length; j++) {
                            let obj = {};
                            obj.pid = pid[j];
                            obj.speed = speed[j];
                            obj.time = time[j];
                            this.tourList[i].push(obj);
                        }
                    }
                    if (parseInt(info[`tourplaystatus.${i}`]) == 1 && info.isfisheye == '0') {
                        this.playTour = i + 1;
                    }
                    if (this.playTour && !this.tourTimerFlag) {
                        this.tourTimerFlag = true;
                        this.tourTimer = setTimeout( () => {
                            this.update_tour_status();
                        }, 1000);
                    }
                }
                for (let i=0; i<this.FISH_MAX_VIEW_NUM; i++) {
                    this.fishTourList.push([]);
                    for (let j=0; j<this.PATROL_NUM; j++) {
                        this.fishTourList[i].push([]);
                        if (info[`fishtourset${i}.${j}`]) {
                            let arr = info[`fishtourset${i}.${j}`].split(';');
                            if (arr == '')
                                continue;
                            let pid = arr[0].split(':');
                            let speed = arr[1].split(':');
                            let time = arr[2].split(':');
                            for (let k=0; k<pid.length; k++) {
                                let obj = {};
                                obj.pid = pid[k];
                                obj.speed = speed[k];
                                obj.time = time[k];
                                this.fishTourList[i][j].push(obj);
                            }
                        }
                    }
                    this.fishAutoScan[i] = parseInt(info[`fishautoscan.${i}`]);
                }
                this.init_tour();
                console.log('this.tourArr:', this.tourArr);
                for (let i=0; i<this.PATTERN_NUM; i++) {
                    // let obj = {};
                    this.patternArr[i].record = parseInt(info[`patternrecord.${i}`]);
                    this.patternArr[i].exist = parseInt(info[`patternstatus.${i}`]);
                    if (parseInt(info[`patternplaystatus.${i}`]))
                        this.playPattern = i + 1;
                }

                if (info.isfisheye == '1') {
                    this.build_display_arr_by_install(this.fishInstall);
                    this.change_tour_data(this.fishDisplay, 0);
                    this.fishSubEnable = [info['fish1Osubenable.1'], info['fish1Psubenable.3'], info['fish2Psubenable.5'], info['fish4Rsubenable.7'], info['fish1O3Rsubenable.11'], info['fish1P3Rsubenable.15']];
                    this.init_fish_view();
                    // let correct = ASTGUI.cookies.getCookie("fisheyeCorrectionModel");
                    if (this.fishCorrect == fishCorrectMode.JOINT_VIDEO)
                        this.showFishWindow = false;
                    
                    this.osdZmTime = info.osdzmtime;
                    this.osdMdTime = info.osdmdtime;
                    this.osdPatrolTime = info.osdpatroltime;
                    this.osdScanTime = info.osdscantime;
                } else
                    this.showFishWindow = false;

                this.mirctrl = parseInt(info.mirctrl);
                
                var_start(this.var_callback);

                this.radarEnable = parseInt(info.radarenable) ? true : false;

                this.vcaenable = parseInt(info.vcaenable);
                this.faceMSCEnable = parseInt(info.facemscenable) == 1 ? true : false;

                this.StartAlarmWorker();
                //
                this.patrolRecover = parseInt(info.patrolrecoveryenable) ? true : false;
                if (this.patrolRecover || this.patrolMode == 1) {
                    this.patrolTimer = setTimeout( () => {
                        this.update_patrol_status();
                    }, 3000);
                    if (!this.tourTimerFlag) {
                        this.tourTimerFlag = true;
                        this.tourTimer = setTimeout( () => {
                            this.update_tour_status();
                        }, 1000);
                    }
                }
            start_online_heartbeat();
            ASTGUI.cookies.setLoginCookie('isHome','1');

            let connectFun = this.lpr_connect;
            this.webSocketObject = function (wsUrl, onmessageCallback, onerrorCallback) {
                let that = this;
                that.pingPong = "";
                let socket;
                if (!window.WebSocket) {
                    window.WebSocket = window.MozWebSocket;
                }
                if (window.WebSocket) {
                    socket = new WebSocket(wsUrl);
                    socket.onmessage = onmessageCallback;
                    socket.onopen = function (event) {
                        console.log("连接开启!");
                    };
                    socket.onclose = function (event) {
                        console.log("连接被关闭");
                        if (this.lprSocket != null) {
                            socket.close();
                        }
                        if (this.isLoaded) {    // destroy don't run this
                            connectFun();
                        }
                    };
                    socket.onerror = onerrorCallback;
                    return socket;
                } else {
                    alert("你的浏览器不支持 WebSocket！");
                }

                this.send = function (message) {
                    if (!window.WebSocket) {
                        return;
                    }
                    if (socket.readyState == WebSocket.OPEN) {
                        socket.send(message);
                    } else {
                        alert("连接没有开启.");
                    }
                }
            };
            this.webSocketObject.prototype.heartCheck = function () {
                var that=this;
                that.pingPong = 'ping'; // ws的心跳机制状态值
                that.pingInterval = setInterval (function () {
                    if (that.ws.readyState === 1) {
                        that.ws.send('ping'); // 客户端发送ping
                    }
                }, 10000);
                that.pongInterval = setInterval(function () {
                    that.pingPong = false;
                    if (that.pingPong === 'ping') {
                        clearInterval(that.pingInterval);
                        clearInterval(that.pongInterval);
                        connectFun(); // 没有返回pong 重启webSocket
                    }
                    console.log('返回pong')
                    that.pingPong = 'ping'
                }, 20000);
            };
            document.addEventListener('mousedown', hide_right_menu);
        },
	},
    mounted() {
		window.show_lpr_img = this.show_lpr_img;
        window.show_lpr_list = this.show_lpr_list;
	},
    async created () {
        this.isSupportPtzSettings = is_support_ptz_settings() || is_support_fisheye_settings()
        this.isSupportPtzCtrl = is_support_ptz_control()
        this.isForbidPtzCtrlAndSettings = !this.isSupportPtzSettings && !this.isSupportPtzCtrl
        this.isAnony = is_anony()
        if (false == is_anony()) {
            if (!is_support_rtsp_access()) {
                this.protocolOptions = this.protocolOptions.filter(item => item.value == 2)
            }
            if (!is_support_live_view()) {
                this.protocolOptions = this.protocolOptions.filter(item => item.value != 2)
            }
            this.liveVideoPri = !IsForbidLiveViewFeature()
        }
        await this.get_config_info();
        get_sys_info(this.sys_info_cb);
    },
    beforeDestroy() {
        console.log('destory')
        this.hasDestroy = true;
        this.alarmTimerFlag = false;
        clearTimeout(this.alarmTimer);
        clearTimeout(this.facePointTimer);
        this.isFaceMode = false;
        clearTimeout(this.facePicTimer);
        this.drawDataTimerFlag = false;
        clearTimeout(this.drawDataTimer);
        this.autoTrackTimerFlag = false;
        clearTimeout(this.autoTrackTimer);
        this.tourTimerFlag = false;
        clearTimeout(this.tourTimer);
        this.fishTourTimerFlag = false;
        clearTimeout(this.fishTourTimer);
        clearTimeout(this.playActiveTimer);
        clearTimeout(this.fishScaleTimer);
        clearTimeout(this.overLayTimer);
        clearTimeout(this.switchLoadingTimer);
        clearTimeout(this.subSmartTimer);
        clearTimeout(this.lprLayTimer);
        clearTimeout(this.faceLayTimer);
        clearTimeout(this.disableFocusTimer)
        this.drawTimerFlag = false;
        clearTimeout(this.drawTimer);
        this.bPtzTimer = 0;
        clearTimeout(this.ptzTimer);
        this.getStateFlag = false;
        clearTimeout(this.getStateTimer);
        this.hide_all_lines();
        playerApi.MsSetPolygonMap(0, 1, '-1:-1:-1:-1:-1:-1:-1:-1:-1:-1', '-1:-1:-1:-1:-1:-1:-1:-1:-1:-1');
        playerApi.MsSetCoutLineData("-1:-1:-1:-1;-1:-1:-1:-1;-1:-1:-1:-1;-1:-1:-1:-1;");
        playerApi.MsSetMotionMap(1,'0');
        this.draw_liveview_rect(0, '0:0:0:0:0:0:');
        playerApi.MsSetPersonRectRegion(1, "0:0:0:0:0:0");
        playerApi.MsSetPersonRectWithTime(1, "");
        playerApi.MsSetRegionAlarm("0:0:-1:-1:-1:-1");
        unInitPlayer(null);
        document.removeEventListener('mousedown', hide_right_menu);
        unregisterIeFun(0);
        unregisterIeFun(1);
        unregisterIeFun(2);
        unregisterIeFun(3);
        this.isLoaded = false;
        if (this.lprSocket != null) {
            this.lprSocket.close();
        }
    },
    watch: {
        $route: {
            handler: function (val, oldVal) {
                console.log(val);
                if (val.path.indexOf('liveview/lpr') != -1) {
                    // location.reload();
                    this.isLprMode = false;
                } else {
                    this.isLprMode = true;
                }
                this.change_lpr_mode();
            }
        },
        "$store.state.app.autoScanStatus": {
            immediate: true,
            handler: function(newVal, oldVal) {
                this.autoScanStatus = newVal
            }
        }
    }
}
</script>
<style lang="scss" scoped>
$backgroudCC: #ccc;
$iconClass: '[class^="icon-"]';
$hoverColor: #16a1fe;
$clickColor: #0b74d1;
$iconColor: #707070;
$iconSize: 24px;
$btnBgColor: $hoverColor;
$btnHoverBgColor: #50b9ff;
$btnClickBgColor: $clickColor;
$dialogHearColor: #43aef6;
.livePage {
	display: flex;
	flex: 1 1 auto;
	margin: 5px;
	background: $backgroudCC;
	border-radius: 4px;
    overflow: hidden;
    width: calc(100% - 68px);
    .isActive {
        color: $hoverColor
    }
	.videoPanel {
		flex: 1 1 auto;
		display: flex;
		flex-flow: column;
		background-color: #f3f3f3;
		border-radius: 4px;
        overflow: hidden;
        width: 100%;
        position: relative;
        min-width: 660px;
		.topBar,
		.btnBar {
			height: 46px;
			display: flex;
			padding: 0 18px;
			align-items: center;
			justify-content: space-between;
			font-size: 20px;
            flex-shrink: 0;
			#{$iconClass}::before {
				font-size: 22px;
				vertical-align: top;
			}
			.select-div {
				color: #363636;
				/deep/ .txt-div {
					font-size: 14px;
				}
			}
			.leftBar {
				display: flex;
				align-items: center;
				.select-div {
					margin-right: 24px;
				}
                .stream-select {
                    width: 150px;
                }
                .protocol-select {
                    width: 60px;
                }
                .smooth-select {
                    width: 120px;
                }
                .overlay-select {
                    width: 200px;
                }
                .face-select {
                    width: 180px;
                }
                .lpr-select {
                    width: 180px;
                }
			}
			.rightImg {
				#{$iconClass} {
					font-size: 22px;
					color: red;
					margin-left: 20px;
                    cursor: default;
				}
                .lpr-black {
                    color: #F00;
                }
                .lpr-white {
                    color: #06A000;
                }
                .lpr-visitor {
                    color: #FFD008;
                }
			}
			.btnLeft {
                .output-button {
                    border: 0px solid #dcdfe6;
                    background-color: #f3f3f3;
                }
                .in-effect {
                    color: #16a1fe;
                }
                i {
                    cursor: pointer;
                }
            }
			.btnRight {
				display: flex;
				align-items: center;
				i {
					font-size: 22px;
					cursor: pointer;
					color: $iconColor;
				}
				i:hover {
					color: $hoverColor;
				}
				i:active {
					color: $clickColor;
				}
                .in-effect {
                    color: #16a1fe;
                }
			}
			.btnRight i#{$iconClass}::before {
				display: inline-block;
				font-size: 24px;
				margin-top: 4px;
			}
			.btnLeft {
				i {
					margin: 0;
					margin-right: 20px;
				}
				i#{$iconClass}::before {
					font-size: $iconSize;
				}
			}
		}

		.canvasBox {
            display: flex;
            flex-flow: column;
			flex: 1 1 auto;
			background: #aeaeae;
			text-align: center;
            position: relative;
            overflow: hidden;
            .origin {
                overflow: auto;
                display: -ms-grid !important;
            }
            .video-box {
                display: flex;
                flex: 1 1 auto;
                align-items: center;
                flex-direction: column;
                -ms-flex-direction: row !important;
                justify-content: center;
                #VideoAxRoom {
                    height: 100%;
                    position: relative;
                    text-align: center;
                    background: #aeaeae;
                    margin: 0 auto;
                    top: 0px;
                    //flex: 1 1 auto;
                    /deep/ .Playerplugin {
                        width: 100%;
                        height: 100%;
                        position: absolute;
                    }
                }
                .lpr-info {
                    //width: 50%;
                    background: #d8d8d8;
                    display: flex;
                    flex-flow: column;
                    overflow: hidden;
                    height: 100%;
                    -ms-grid-column: 2;
                    img {
                        flex: 1 1 auto;
                        height: calc(100% - 77px);
                    }
                    img[src=""], img:not([src]) {
                        opacity: 0;
                    }
                    .lpr-info-txt {
                        padding: 10px;
                        border-top: 1px solid #ccc;
                        background: #f3f3f3;
                        display: flex;
                        align-items: center;
                        .lpr-plate-box {
                            height: 55px;
                            width: 32%;
                            display: inline-block;
                            span {
                                width: 100%;
                                text-align: center;
                            }
                            .lpr-plate {
                                font-size: 2rem;
                                display: inline-block;
                                width: 100%;
                            }
                        }
                        .lpr-plate-info {
                            flex: 1 1 auto;
                            display: inline-block;
                            width: 68%;
                        }
                        span {
                            width: 30%;
                            display: inline-block;
                            text-align: left;
                            padding: 2px;
                            font-size: 14px;
                        }
                        label {
                            margin-left: 6px;
                            font-weight: bold;
                        }
                    }
                }
                .videoTips {
                    width: 100%;
                    height: 100%;
                    background-color: #000;
                    /deep/ .videoTipTxt {
                        position: relative;
                        top: 47%;
                        font-size: 24px;
                    }
                }
            }
            .lpr-list {
                height: 33%;
                background: #fff;
                overflow: auto;
                .el-table {
                    border: none;
                    .cell{
                        overflow: visible;
                    }
                }
                table {
                    border-collapse: collapse;
                    th{
                        height: 36px;
                        background: #53b5f3;
                        color: white;
                        white-space: nowrap;
                    }
                    /deep/ {
                        tr.el-table__row {height: 30px;}
                        td {overflow: visible;}
                        img {
                            display: inline-block;
                            width: 80px;
                            max-height: 23px
                        }
                        a {
                            color:#2c3e50;
                            font-size: 18px;
                            white-space: nowrap;
                        }
                        a:hover{color:#16a1fe;}
                    }
                }
                
            }
            .lpr-big-img {
                position: absolute;
            }
            
		}
	}
    .triggerRight {
		display: inline-block;
		width: 15px;
		color: white;
		background-color: #000000;
		border-radius: 4px;
		opacity: 0.4;
		position: absolute;
        top:48%;
		right: 0;
		height: 80px;
        line-height: 80px;
        cursor: pointer;
        z-index: 98;
	}
    .rotate {
        transform: rotate(180deg);
    }
	.leftPanel {
		width: 248px;
		background: #efefef;
		border-radius: 4px;
		margin-left: 1px;
		position: relative;
		display: flex;
		align-items: center;
        flex-shrink: 0;
		.tabsBox {
			width: 100%;
			display: flex;
			flex-flow: column;
			height: 100%;
			background: none;
			border: none;
			box-shadow: none;
			#{$iconClass} {
				font-size: $iconSize;
				cursor: pointer;
				color: $iconColor;
			}
            .in-effect {
                color: $hoverColor;
            }
			/deep/ {
				.el-tab-pane {
					height: 100%;
				}
				.el-tabs__header {
					background: transparent;
				}
				.el-tabs__item {
					height: 46px !important;
					border-bottom: 1px solid #ccc;
					background: #efefef;
					width: 82px;
					text-align: center;
					margin-top: 1px;
					margin-left: 0;
				}
				.el-tabs__item.is-active {
					background: $hoverColor;
					#{$iconClass} {
						color: white;
					}
				}
				.el-tabs__nav-wrap,
				.el-tabs__nav-scroll {
					overflow: inherit;
					.el-tabs__nav {
						display: flex;
					}
				}
				.el-tabs__content {
					flex: 1 1 auto;
				}
				.is-active::after {
					content: "";
					border-width: 7px 7px 0;
					border-style: solid;
					border-color: #16a1fe transparent transparent;
					position: absolute;
					bottom: -7px;
					left: 35px;
				}
				.el-slider {
					flex: 1 1 auto;
					margin-left: 10px;
				}
                .fish-is-disabled {
                    color: #c0c4cc;
                    cursor: not-allowed;
                }
                .fish-is-disabled:hover {
                    color: #c0c4cc !important;; 
                    cursor: not-allowed;
                }
			}
			.fishPanel {
				h3 {
					border-left: 2px solid #16a1fe;
					padding-left: 10px;
					font-weight: normal;
					font-size: 14px;
				}
				> div {
					padding: 10px 0;
					i {
						display: inline-block;
						margin: 10px 13px 0;
					}
					i:hover {
						color: $hoverColor;
					}
					i:active {
						color: $clickColor;
					}
				}
			}
			.flexBox {
				display: flex;
				flex-wrap: wrap;
				padding: 0 25px;
				text-align: center;
				margin: 0 auto;
			}
			.listBox {
				margin-top: 10px;
				#{$iconClass} {
					font-size: 20px;
					margin-top: 0;
				}
				li {
					height: 26px;
					line-height: 26px;
					cursor: pointer;
					font-size: 14px;
				}
				li:hover {
					background: #dfdfdf;
				}
			}
			.imgPanel {
				text-align: center;
				.imgInfo li {
					display: flex;
					align-items: center;
					margin-bottom: 10px;
					i {
						font-size: $iconSize;
					}
				}
				.imgInfo .btn-box {
					display: inline-block;
				}
			}
			.ptzPanel {
				height: 100%;
				display: flex;
				flex-flow: column;
				.ptz-list {
					flex: 1 1 auto;
					display: flex;
					flex-flow: column;
					overflow: hidden;
					.infinite-list {
						height: 100%;
                        scrollbar-width: thin;
		                scrollbar-color: rgba(144, 147, 153, 0.3) transparent;
					}
					[class*="icon-"] {
						font-family: "iconfont" !important;
						padding: 3px;
						margin: 0;
						background: transparent;
						border: none;
						font-size: 20px;
					}
					li {
						justify-content: flex-start;
					}
                    label {
                        margin-right: 10px;
                    }
                    .tour-name,.pattern-name {
                        width: 86px;
                    }
                    .preset-list {
                        .preset-name {
                            width: 90px;
                            margin-right: 7px;
                            overflow: hidden;
                            white-space: nowrap;
                            text-overflow: ellipsis;
                        }
                        span { 
                            width: 88px;
                            text-align: right;
                        }
                        i:hover {
                            color: $hoverColor;
                        }
                        i:active {
                            color: $clickColor;
                        }
                        /deep/ .el-input {
                            max-width: 80px;
                            .el-input__inner {
                                padding: 0 5px;
                            }
                        }
                    }
                    .tour-list {
                        span {
                            width: 78px;
                            text-align: right;
                        }
                    }
					.tour-header {
						height: 32px;
						line-height: 32px;
						background: #fff;
						border-radius: 4px;
						padding: 0 5px;
						display: flex;
						justify-content: space-between;
						margin-bottom: 10px;
						font-size: 14px;
						label {
							position: relative;
						}
						label:before {
							content: "";
							width: 2px;
							background: #16a1fe;
							position: absolute;
							left: -5px;
							height: 50%;
							top: 25%;
						}
                        .el-button {
                            height: 32px !important;
                        }
					}
					.tour-set {
						margin: 10px auto;
						font-size: 14px;
                        overflow-y: auto;
                        max-height: 356px;
						li {
							justify-content: flex-end;
						}
                        li:first-child {
							margin-bottom: 5px;
						}
                        label {
                            margin-right: 0;
                        }
						.preset,
						.speed {
							width: 64px;
							margin-left: 5px;
                            /deep/.el-input__suffix {
                                right: -5px !important;
                            }
						}
						.list-index {
							width: 20px;
						}
						.time {
							width: 50px;
							margin-left: 5px;
                            margin-right: 0;
						}
                        .selected-li {
                            background-color: #CCC;
                        }
						.is-first {
							position: relative;
							overflow: visible;
							.el-form-item__error {
								left: 150px;
								width: 50px;
								min-width: unset !important;
							}
						}
					}

					/deep/ .el-input,
					.el-select {
						max-width: 86px;
						height: 26px;
						line-height: 26px;
						.el-input__inner,
						.el-input__icon {
							height: 26px;
							line-height: 26px;
							border-radius: 1px;
                            padding: 0 8px;
						}
                        .el-icon-arrow-up:before {
                            font-family: 'iconfont' !important;
                            content: '\e817';
                        }
					}
					.btn-bar {
						text-align: center;
					}
				}
			}
			.ptz-ctrl {
				display: flex;
				flex-wrap: wrap;
				width: 140px;
				background: #fff;
				margin: 0 auto;
				border: 1px solid #d5d5d5;
				justify-content: space-around;
				li {
					padding: 10px;
                    user-select: none;
				}
				li:hover {
					color: $hoverColor;
				}
				li:active {
					color: $clickColor;
				}
				.btn-border {
					padding: 5px 15px;
					background: #fff;
					border: 1px solid #d5d5d5;
					margin-bottom: 10px;
					border-radius: 2px;
				}
				.icon-ptzauto {
					border-radius: 50%;
					border: 1px solid;
				}
                li.locked-active:hover {
                    color: $hoverColor;
                }
                li.locked-active:active {
                    color: $hoverColor;
                }
                li.locked-disabled:hover {
                    color: $iconColor;
                }
                li.locked-disabled:active {
                    color: $iconColor;
                }
			}
			.focus-ctrl {
				background: transparent;
				border: none;
			}
			li {
				display: flex;
				align-items: center;
			}
			.ptz-feature {
				display: flex;
				border-top: 1px solid #fff;
				border-bottom: 1px solid #fff;
				flex-wrap: wrap;
				justify-content: center;
				margin: 0 auto;
				padding: 10px;
                flex: 0 0 auto;
				li {
					padding: 10px 15px;
					position: relative;
				}
				li:not(:nth-child(3n + 1))::after {
					content: "";
					display: inline-block;
					width: 1px;
					height: 50%;
					background: #acacac;
					left: 0;
                    top: 11px;
					position: absolute;
				}
				li:hover {
					color: $hoverColor;
				}
				li:active {
					color: $clickColor;
				}
			}
			.ptz-list /deep/ {
				.el-tabs__item {
					width: 72px;
					height: 30px !important;
					line-height: 30px;
					background: #fff;
					border: none;
					border-radius: 4px;
					padding: 0;
				}
				.el-tabs__header {
					margin-bottom: 5px;
				}
				.el-tabs__item.is-active {
					background: $hoverColor;
					#{$iconClass} {
						color: white;
					}
				}
				.is-active::after {
					display: none;
				}
			}
		}
		.msButton {
			font-size: 14px;
		}
	}

	.face-panel {
		width: 230px; //12%
		padding: 4px;
		background-color: #f2f2f2;
		border-radius: 4px 0px 0px 4px;
		margin-right: 1px;
        flex-shrink: 0;
		ul {
			height: 100%;
		}
		li {
			display: flex;
			background-color: #ffffff;
			border: solid 1px #c8c8c8;
			color: #666;
			align-items: center;
			position: relative;
			img {
                min-width: 90px;
				width: 90px;
				height: 136px;
				margin: 5px;
				z-index: 2;
			}
			.face-info {
				margin: 5px 0;
				position: relative;
				flex: 1 1 auto;
				i {
					display: inline-block;
					font-size: 36px;
					width: 50%;
					text-align: center;
					margin: 10px auto;
					color: #858585;
				}
				label {
					font-size: 12px;
					display: block;
				}
			}
			.face-info:before {
				content: "";
				width: 70%;
				height: 1px;
				display: inline-block;
				position: absolute;
				top: 45%;
				left: 15%;
				background: #dfdfdf;
			}
			.face-info:after {
				content: "";
				width: 1px;
				height: 66%;
				display: inline-block;
				position: absolute;
				top: 13%;
				left: 50%;
				background: #dfdfdf;
			}
		}
		li:before {
			content: "";
			width: 94px;
			height: 140px;
			margin: 5px;
			background: linear-gradient(
				130deg,
				#50a7ff,
				rgba(132, 193, 246, 0.2),
				#50a7ff
			);
			position: absolute;
			left: -2px;
			top: -2px;
			z-index: 1;
		}
        #faceMSCTip {
            height: 100%;
            text-align: center;
            color: #888484;
            display: flex;
            align-items: center;
            justify-content: center;
        }
	}

    .image-container {
        width: 840px;
        max-width: 100%;
    }

    /deep/ .el-dialog {
        .el-dialog__body {
            .el-form {
                .el-form-item__label-wrap{
                    margin-left: 0;
                }
                .el-input__inner {
                    width: 240px;
                }
            }
        }
    }
}
.lpr-videoroom {
    height: 67%;
    display: -ms-grid !important;
    -ms-grid-columns: 50% 50%;
    -ms-grid-rows: 100%;
    display:grid !important;
    grid-template-columns: 50% 50%;
    grid-template-rows: 100%;
}
.lpr-videoroom>div{
    display: inline-block;
    flex: 1 1 50% !important;
}
.lpr-videoroom #VideoAxRoom{
    margin:0;
    flex-grow: 1;
}
.lpr-videoroom #MaxConnTips{
    margin:0;
    position: relative;
    line-height: 30 !important;
    /deep/ .videoTipTxt {
        top: 5%;
    }
}
.lpr-videoroom #VideoCtrl{
    width: 100%;
    height: 100%;
}
// add by felix
.el-form {
    padding-right: 2px;
    max-height: 100%;
    overflow-x: hidden;
    overflow-y: auto;
    padding-left: 10px;
    margin-left: -10px;
    scrollbar-width: thin;
    scrollbar-color:rgba(144, 147, 153, .3) transparent;
}
.el-form-item__label-wrap {
    height: 0;
    margin-left: 0px !important;
}
.el-input__icon {
    font-size: 18px !important;
}
.el-input {
    width: 240px;
    margin-right: 14px;
    vertical-align: middle;
}
.el-input--mini {
    font-size: 14px;
}
.el-form-item__content > .el-input--mini > input[readonly],
.cell input[readonly],
textarea[readonly] {
    background-color: #dbdbdb;
    border: solid 1px #b7b7b7;
}
.el-form-item__label {
    position: relative;
    padding: 0;
}
.el-form-item__content {
    padding-left: 12px;
}

.el-form-item.is-required .el-form-item__label:before {
    position: absolute;
    right: -12px;
    font-size: 20px;
    top: 5px;
}
.el-button {
    font-size: 14px;
    padding: 7px 10px;
}
.el-button--primary {
    padding: 7px 30px;
}
.btn-active,
.el-button--primary {
    background: $btnBgColor;
    color: #fff;
}
.el-button--primary:hover {
    background: $btnHoverBgColor;
}
.el-button--primary:active {
    background: $btnClickBgColor;
}
.el-button--primary.is-plain {
    background: white;
    color: $hoverColor;
    padding: 6px 20px;
    border-radius: 2px;
    vertical-align: middle;
}
.el-button--primary.is-plain:hover {
    background: #d5f0fc;
}
.el-button--primary.is-plain:active {
    background: #a8d3e6;
}
.el-button--primary.is-disabled,
.el-button--primary.is-disabled:active,
.el-button--primary.is-disabled:focus,
.el-button--primary.is-disabled:hover {
    background-color: #c0c4cc;
    border-color: #c0c4cc;
    color: white;
}
.face-time {
    text-align: center;
}

.face-detail-time {
    text-align: left;
    margin-left: 38px;
    margin-bottom: 10px;;
}

.attribute ul li{
    float: left;
    width: 200px;
    line-height: 28px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    word-wrap: inherit;
    text-align: left;
}

.facePic {
    float: left;
    width: 80px;
    height: 80px;
    margin-right: 80px;
    text-align: center;
    vertical-align: middle;
    background-size: 100% 100%;
}

.backgroundPic {
    float: left;
    width: 550px;
    height: 300px;
    margin-top: 10px;
    margin-left: 40px;
    background-size: 100% 100%;
}
.face-bottom {
    margin-bottom: 30px;
}

.disableClick {
    pointer-events: none;
}
.icon-alarm-1 {
    position: relative;
    #output-alarm-popover {
        position: absolute;
        padding-top: 20px;
        padding-bottom: 20px;
        width: 200px;
        left: -75px;
        bottom: 30px;
        background: #fff;
        border: 1px solid #d5d5d5;
        border-radius: 3px;
        text-align: center;
        z-index: 9999;
        .output-span {
            display: inline-block;
            width: 60px;
            word-break: break-word;
            text-align: left;
            font-size: 14px;
            line-height: 1.5;
            color: #606266;
        }
	}

    #output-alarm-popover::after {
        content: '';
        width: 0;
        height: 0;
        font-size: 0;
        overflow: hidden;
        display: block;
        border-width: 6px;
        border-color: #fff transparent transparent transparent;
        border-style: solid dashed dashed dashed;
        position: absolute;
        bottom: -12px;
        left: 40%;
        pointer-events: none;
    }
}

@media screen and (max-width:1121px) {
	.livePage{
		overflow-x: visible !important; 
        min-width: 1037px;
	}
}

/deep/ .lpr-add-disable {
    color: #c0c4cc !important;
    cursor: not-allowed;
}
/deep/ .lpr-add-disable:hover {
    color: #c0c4cc !important;; 
    cursor: not-allowed;
}
		
</style>
