<template>
    <div id="evaluate-detail"
         :class="{'editable': editable}"
         element-loading-spinner="iconfont wui-Loading"
         v-loading.fullscreen="loading">
        <!--评分-->
        <header-wrap class="operation"
                     :allowBack="true"
                     :backFn="toIndex"
                     :ifFixed="true">
            <div slot="title">
                <!--评分-->{{_sn('P_YS_OA_XTLCZX_0000030728')}}</div>
            <div slot="operation">
                <Progress v-if="baseList.schememode === 20"
                          :textArr="halfstepArr"
                          :active="activeStep"
                          :size="24">
                </Progress>
                <Progress v-else
                          :textArr="stepArr"
                          :active="activeStep"
                          :size="24">
                </Progress>
            </div>
            <el-row slot="content"
                    type="flex"
                    align="middle"
                    class="rep-card">
                <div style="float:left; margin-right: 16px;">
                    <div class="carge"
                         :style="{'background-color': baseList.nameBack,'background-image': 'url(\'' +( baseList.photo || 'static/images/head-portrait.png') + '\')'}">
                    </div>
                    <div class="switch"
                         v-show="fromList">
                        <div @click="changeP(-1)"
                             :style="{opacity: parseInt(ind)=== 0 ? 0 : 1,cursor:parseInt(ind)=== 0 ? 'auto' : 'pointer'}">
                            <!--上一人-->
                            <el-tooltip :content="_sn('P_YS_HR_HRJXF_0000059678')"
                                        :disabled="parseInt(ind)=== 0"><i class="iconfont wui-left_bold"></i>
                            </el-tooltip>
                        </div>
                        <div @click="changeP(1)"
                             :style="{opacity: parseInt(ind) === parseInt(len) - 1 ? 0 : 1,cursor:parseInt(ind) === parseInt(len) - 1 ? 'auto' : 'pointer'}">
                            <!--下一人-->
                            <el-tooltip :content="_sn('P_YS_HR_HRJXF_0000059327')"
                                        :disabled="parseInt(ind) === parseInt(len) - 1"><i
                                   class="iconfont wui-right_bold"></i></el-tooltip>
                        </div>
                    </div>
                </div>
                <div style="line-height: 1.6;display: inline-block;width:100%">
                    <span class="card-tit">{{ baseList.staffname }}</span>
                    <span class="card-info">{{ baseList.deptname }}</span>
                    <div style="background-color: #d0d0d0;height:1px; width:100%;margin-top: 7px;margin-bottom:7px;">
                    </div>
                    <div class="rep-nav"
                         id="bar">
                        <el-row type="flex">
                            <el-col :span="12">
                                <div class="score-part">
                                    <!--我的评分-->
                                    <div>
                                        <p class="tips"
                                           v-show="schememode !== 20">{{_sn('P_YS_HR_HRJXF_0000059472')}}</p>
                                        <p style="font-size: 20px;height:20px;"
                                           v-show="schememode !== 20">{{score}}</p>
                                    </div>
                                    <!--前轮总分-->
                                    <div class="wheel"
                                         v-if="turn != '1' && schememode !== 20"
                                         @click="toPreTurn">
                                        <p class="tips">{{_sn('P_YS_HR_HRJXF_0000059563')}}</p>
                                        <p class="preturn"
                                           style="font-size: 20px;height:20px">{{lastScore || ' '}}</p>
                                    </div>
                                </div>
                            </el-col>
                            <el-col :span="12"
                                    class="right">
                                <!--提交-->
                                <el-button type="primary"
                                           @click="save('submit')"
                                           v-if="editable && !(selfRating && radioStrips)"
                                           v-loading.fullscreen.lock="fullscreenLoading"
                                           element-loading-spinner="iconfont wui-Loading">{{_sn('P_YS_FED_UCFBASEDOC_0000024984')}}
                                </el-button>
                                <el-button type="primary"
                                           @click="sendOf"
                                           v-if="editable && selfRating && radioStrips"
                                           v-loading.fullscreen.lock="fullscreenLoading"
                                           element-loading-spinner="iconfont wui-Loading">{{_sn('P_YS_FED_UCFBASEDOC_0000024984')}}
                                </el-button>
                                <!--保存-->
                                <el-button type="info"
                                           @click="save"
                                           v-show="editable">{{_sn('P_YS_PF_GZTSYS_0000013477')}}</el-button>
                                <!-- 中期回顾 -->
                                <el-button @click="toMidReview"
                                           type="info"
                                           v-show="isallowmtr">{{_sn('P_YS_HR_HRJXF_0000130274')}}</el-button>
                                <!--收回-->
                                <!--复制前轮评分-->
                                <el-button type="info"
                                           v-if="parseInt(turn) !== 1 && editable && schememode !== 20"
                                           @click="getlastTurn()">{{_sn('P_YS_HR_HRJXF_0000059561')}}</el-button>
                            </el-col>
                        </el-row>
                    </div>
                </div>
            </el-row>
        </header-wrap>
        <el-row v-if="editable && schememode === 4"
                id="offends"
                class="offend">
            <!-- 合规性考核 -->
            <el-col :span="3">{{_sn('P_YS_HR_HRJXF_0000130289')}}</el-col>
            <el-col :span="15"
                    style="padding: 0 12px;">
                <div>
                    <!-- 是否遵守《员工商业行为守则》及公司其他规章制度的要求 -->
                    {{_sn('P_YS_HR_HRJXF_0000130210')}}<div class="contraband"
                         @click="showStrips">{{_sn('P_YS_HR_HRJXF_0000130284')}}</div>
                </div>
                <div style="line-height: 2;">
                    <!-- 未遵守 -->
                    <el-radio v-model="radioStrips"
                              @change="offend"
                              :label=true>{{_sn('P_YS_HR_HRJXF_0000130251')}}</el-radio>
                    <!-- 遵守 -->
                    <el-radio v-model="radioStrips"
                              @change="offend"
                              :label=false>{{_sn('P_YS_HR_HRJXF_0000130224')}}</el-radio>
                </div>
            </el-col>
        </el-row>
        <el-container>
            <el-main>
                <el-collapse>
                <div class="scoreName">
                    <!-- 绩效指标 -->
                    <span>{{_sn('P_YS_HR_HRJXF_0000130227')}}</span>
                </div>
                <div class="rep-type"
                     v-for="(val, index) in listData"
                     :key="index">
                     
                    <div class="rep-bar">
                        <i class="bar-tip"></i>
                        <el-row type="flex"
                                align="middle">
                            <div class="bar-title"><span>{{ val.parentname }}</span></div>
                            <div v-if="!baseList.isdoubleweight && scoreway === 1 && val.numtotal !== ''"
                                 class="bar-per percent">{{ val.numtotal.toFixed(0) }}%</div>
                            <div v-if="baseList.isdoubleweight && scoreway === 1"
                                 class="bar-per">{{ val.indiclass_weight }}%</div>
                        </el-row>
                        <!-- <el-row class="nameType">工作目标</el-row> -->
                    </div>
                     <el-collapse-item>
                    <el-row>
                        <div class="subscribe"
                             v-if="schememode === 4">
                            <!-- 评分说明 -->
                            <p>{{_sn('P_YS_HR_HRJXF_0000059722')}}:</p>
                            <!-- <p>1、采用评分制，对照每条价值观的行为标准，由低至高，做到价值观对应分值的行为标准，即获得该价值观的对应分数；</p> -->
                            <p>{{_sn('P_YS_HR_HRJXF_0000130242')}}</p>
                            <!-- <p>2、每一条若只做到了部分，可打0.5分；</p> -->
                            <p>{{_sn('P_YS_HR_HRJXF_0000130261')}}</p>
                            <!-- <p>3、每一条价值观评1分（含）以下，或者3.5（含）分以上，需要提供事例举证；</p> -->
                            <p>{{_sn('P_YS_HR_HRJXF_0000130218')}}</p>
                            <!-- <p>4、价值观评价暂不实行负分，可以有0分；</p> -->
                            <p>{{_sn('P_YS_HR_HRJXF_0000130253')}}</p>
                        </div>
                    </el-row>
                    <template v-for="(value, ind) in val.child"
                                >
                        <indi-goal :key="ind" :goalname="value.objectivename"></indi-goal>
                        <div class="rep-box">
                            <div class="rep-item boxes"
                                v-for="(item, idx) in value.objChild"
                                :key="idx">
                                <el-row>
                                    <el-col :span="7">指标</el-col>
                                    <el-col :span="7">评分说明</el-col>
                                    <el-col :span="3">指标权重</el-col>
                                    <el-col :span="3">自评分</el-col>
                                    <el-col :span="3">实际完成值</el-col>
                                </el-row>
                                <el-row>
                                    <el-col :span="7" class="item-title"><span class="i-title">{{ item.indiname }} </span></el-col> 
                                    <el-col :span="7">评分说明</el-col>
                                    <el-col :span="3"><span>{{item.act_weight.toFixed(0)}}%</span></el-col>      
                                        <el-col :span="3">
                                            <div style="display: inline-block"
                                                class="score-part">
                                                <el-select v-if="item.scoreRule && item.scoreRule.ruleType == 1 && editable"
                                                        :disabled="item.scoringmethod > 1"
                                                        class="selectGrade"
                                                        value-key="text"
                                                        @change="selectChange($event, tempListData[index].child[ind].objChild[idx].oappraiseData['008'], index, ind, idx)"
                                                        v-model="tempListData[index].child[ind].objChild[idx].oappraiseData['008']">
                                                    <el-option v-for="(options, op) in item.scoreRule['ogradeItem']"
                                                            :key="options.text"
                                                            :label="options.text"
                                                            :value="options"></el-option>
                                                </el-select>
                                                <el-input-number v-else-if="item.scoreRule && item.scoreRule.ruletype == 0 && editable"
                                                                :disabled="item.scoringmethod > 1"
                                                                v-model="item.oappraiseData['008'].numvalue"
                                                                :placeholder="translate('P_YS_HR_HRJXF_0000059747',[item.scoreRule.lowerlimit,item.scoreRule.upperlimit])"
                                                                :controls="false"
                                                                :precision="item.scoreRule.decimaldigits"
                                                                @change="clearNum">
                                                </el-input-number>
                                                <el-select v-else-if="scoreLimit.ruletype == 1 && editable && !item.scoreRule"
                                                        v-model="item.oappraiseData['008'].gradeitemid"
                                                        class="selectGrade"
                                                        :disabled="hasStrips || item.scoringmethod > 1">
                                                    <el-option v-for="item in scoreLimit.ogradeItem"
                                                            :key="item.id"
                                                            :label="item.gradeitemname"
                                                            :value="item.id">
                                                    </el-option>
                                                </el-select>
                                                <el-input-number v-else-if="editable"
                                                                :disabled="hasStrips || item.scoringmethod > 1"
                                                                v-model="item.oappraiseData['008'].numvalue"
                                                                :placeholder="handlerPlaceholder(item)"
                                                                :controls="false"
                                                                :precision="scoreLimit.decimaldigits"
                                                                @change="clearNum">
                                                </el-input-number>
                                                <span
                                                    v-else-if="scoreLimit.ruletype == 1">{{item.oappraiseData['008'].text}}</span>
                                                <span
                                                    v-else>{{item.oappraiseData['008'].text ? item.oappraiseData['008'].text : item.oappraiseData['008'].numvalue}}</span>
                                            </div>
                                        </el-col>
                                        <el-col :span="3"
                                                class="unit-main">
                                            <!--<p v-if="!editable">-->
                                            <pre v-if="!editable"
                                                class="e-pre"
                                                style="">{{ item.osheetData['006'].text || '&nbsp;' }}</pre>
                                            <!--</p>-->
                                            <mix-textarea v-else
                                                        style="position: relative"
                                                        resize="none"
                                                        :disabled="item.scoringmethod > 1 && item.scoringmethod !== 3"
                                                        type="textarea"
                                                        :rows="4"
                                                        :maxlength="1000"
                                                        v-model="item.osheetData['006'].text">
                                            </mix-textarea>
                                        </el-col>                         
                                </el-row>

                                
                            </div>
                        </div>
                    </template>
                     </el-collapse-item>
                </div>
                <div class="rep-type comp">
                    <div class="rep-bar ">
                        <!-- <i class="bar-tip"></i> -->
                        <el-row type="flex"
                                align="middle">
                            <!--综合评价-->
                            <el-col :span="12"
                                    class="bar-title "><span>{{_sn('P_YS_HR_HRJXF_0000059742')}}</span>
                                <!-- 查看人才画像 -->
                                <span v-if="schememode !== 4"
                                      @click="showTalent"
                                      class="talent-link">{{_sn("P_YS_HR_HRJXF_0000059350")}}</span>
                            </el-col>
                        </el-row>
                    </div>
                    <div class="rep-box">
                        <div class="complex boxes">
                            <!--各轮考核评价-->
                            <!-- <p class="item-title">{{_sn('P_YS_HR_HRJXF_0000059814')}}</p> -->
                            <el-row v-for="(item, index) in overviewData"
                                    :key="index"
                                    v-show="index!== 0 || !editable"
                                    class="complex-scan">
                                <el-col :span="3">
                                    <!--第-->
                                    <span>{{_sn('P_YS_FED_EXAMP_0000020233')}}</span>
                                    <span>{{item.turn}}</span>
                                    <!--轮-->
                                    <span>{{`${_sn('P_YS_HR_HRJXF_0000059852')}: ${item.appraisername}`}}</span>
                                </el-col>
                                <!--评分-->
                                <el-col :span="6"
                                        v-show="schememode !== 20">
                                    {{_sn('P_YS_HR_HRJXF_0000059443')}}
                                    <span style="color:#8E9EE5">{{item.text || item.score}}</span>
                                </el-col>
                                <el-col :span="15">
                                    {{item.comment}}
                                </el-col>
                            </el-row>
                            <el-row v-if="isadjust">
                                <el-col>
                                    <span style="line-height:32px; padding-right: 10px">{{adjustscorename}}</span>
                                    <!-- 请输入调整分 -->
                                    <el-input-number v-model="adjust"
                                                     v-if="editable"
                                                     style="width: 160px;"
                                                     
                                                     :min="0"
                                                     :max="10"
                                                     v-limitNum
                                                     :placeholder="_sn('P_YS_HR_HRJXF_0000130225')"
                                                     :controls="false">
                                    </el-input-number>
                                    <span style="color:#8E9EE5"
                                          v-else>{{adjust ? adjust : ''}}</span>
                                </el-col>
                            </el-row>
                            <el-row v-if="editable">
                                <!--我的评价：-->
                                <el-col :span="3">
                                    <span style="color: #ee2223;"
                                          v-show="hasStrips">*</span>
                                    <span>{{_sn('P_YS_HR_HRJXF_0000059536')}}</span>
                                </el-col>
                                <el-col :span="15">
                                    <mix-textarea resize="none"
                                                  type="textarea"
                                                  :rows="4"
                                                  :maxlength="1000"
                                                  :placeholder="overAllTip"
                                                  v-model="comment">
                                    </mix-textarea>
                                </el-col>
                            </el-row>
                        </div>
                    </div>
                </div>
            </el-collapse>
            </el-main>
        </el-container>
        <transition name="fade">
            <div class="chart-detail exceptClass"
                 id="chart-detail"
                 v-if="talentVisible"
                 v-clickoutside="shutDetail">
                <sideFlow :sheetid="sheetid"
                          :periodid="periodid"
                          :startDate="startDate"
                          :endDate="endDate"
                          ref="TargetTrack"
                          :staffid="staffid"
                          @shutDetail="shutDetail" />
                <!-- :startDate="startDate"   :endDate="endDate" -->
            </div>
        </transition>
        <!-- 前轮评分 -->
        <transition name="fade">
            <div class="wheelFlow"
                 v-if="preturnsVisible"
                 v-clickoutside="closePreTurn">
                <nosewheel-eval v-if="sheetid"
                                :sheetid="sheetid"
                                :turn="turn"
                                :schememode="schememode"
                                :scoreway="scoreway"
                                :closePreTurn="closePreTurn"></nosewheel-eval>
            </div>
        </transition>
        <el-dialog class="iframeDialog"
                   :visible.sync="showIframeDialog"
                   width="80%"
                   height="90%">
            <iframe src="#"
                    name="iframe"
                    id="iframe"
                    ref="iframe"
                    frameborder="0"
                    style="width: 100%;height: 99%;"></iframe>
        </el-dialog>
        <strips @show="showStrips"
                @close="closeStrips"
                ref="strips"
                @offend="offend"></strips>
    </div>
</template>
<script>
import BaseLayout from '@/components/common/base-layout/index'
import HeaderWrap from '@/components/common/header-wrap'
import TitleHeader from '@/components/common/title-header/index'
import { fetchData } from '@/utils/util.js'
import Ref from '@/components/controls/reference'
import MixTextarea from '@/components/common/mix-textarea'
import Progress from '@/components/common/progress/progress'
import merge from 'webpack-merge'
import translate from '@/utils/locale/index.js'
import sideFlow from './talentPortrait'
import Clickoutside from '@/components/controls/clickoutside.js'
import Strips from './strips'
import nosewheelEval from './nosewheelEval'
import { getStorage, removeStorage } from '@/utils/scroll.js'
import IndiGoal from '../common/indi/indigoal' // 目标 项
const { performance } = window.ServiceConfigs
export default {
    components: {
        MixTextarea,
        Ref,
        BaseLayout,
        HeaderWrap,
        TitleHeader,
        Progress,
        sideFlow,
        Strips,
        nosewheelEval,
        IndiGoal
    },
    directives: {
        Clickoutside
    },
    data: function () {
        return {
            showIframeDialog: false,
            hideClass: false,
            modValue: '', // 输入框遮挡参照 输入框value 值
            fendis: false,
            arrBox: [],
            weigData: [],
            dialogweightVisible: false,
            detailShow: false,
            fullscreenLoading: false,
            anditStr: '',
            checkStr: '',
            idx: 0, // 编辑弹框内容索引
            checkVisible: false, // 考核人弹框控制器
            checkData: [], // 考核人弹框控制器
            anditVisible: false, // 审核人弹框控制器
            anditData: [], // 审核人弹框数据
            pk_pe_doc: '', // 暂时值，稍后取消
            sheetid: '',
            pk_pe_paper: '',
            baseList: {}, // 人员基本信息
            listData: [], // 指标列表
            tempListData: [],
            loading: false,
            show: true,
            active: 2, // 步骤条当前值
            topShow: false, // 返回顶部显隐
            topTimer: null, // 返回顶部动画计时器
            dialogVisible: false, // 弹窗显隐
            dialogTitle: this._sn('P_YS_HR_HRPUBF_0000053759'), // 新增考核项
            id: '123', // 人员基本档案ID
            isEdit: true,
            chartLoading: false,
            // 保存指标参数
            targetData: {
                name: { name: '', pk: '' },
                type: { name: '', pk: '' },
                percent: '',
                plan: '',
                standard: '',
                id: '',
                //   childId: '',
                parentId: '',
                parentList: ''
            },
            pk_rel_apr: '',
            // 保存考核方案参数
            formData: {
                period: '',
                type: '',
                percent: ''
            },
            myData: [],
            //
            turn: 1,
            scoreLimit: {},
            score: '',
            lastScore: '',
            chart: null,
            overviewData: [],
            comment: '',
            editable: true,
            flagSave: 0,
            text: '', // 评分说明无009时供界面加载
            radarData: [
                {
                    flag: 0,
                    data: {
                        radarInfo: []
                    }
                }
            ],
            attendData: {},
            periodid: '',
            mas: [],
            ind: 0, // 当前人在方案列表里的顺序
            len: 1, //
            staffList: [],
            lastList: {},
            msg: [],
            changeFlag: {
                getBaseInfo: false,
                getData: false,
                getOverview: false
            },
            chartData: [
                {
                    'flag': 0,
                    'data': {
                        'radarInfo': [
                            { 'percentage': 32, 'characterName': this._sn('P_YS_HR_HRJXF_0000059412') }, // 工作经验
                            { 'percentage': 41, 'characterName': this._sn('P_YS_HR_HRRCPDF_0000071264') }
                        ],
                        'radarName': this._sn('P_YS_HR_HRJXF_0000184353')
                    }
                }
            ],
            indicator: [
                { text: '', max: 120, color: '#DBDBDB' },
                { text: '', max: 120, color: '#DBDBDB' },
                { text: '', max: 120, color: '#DBDBDB' },
                { text: '', max: 120, color: '#DBDBDB' },
                { text: '', max: 120, color: '#DBDBDB' },
                { text: '', max: 120, color: '#DBDBDB' }
            ], // 雷达图维度
            // 指标制定
            // 指标审核
            // 考核评分
            // 考核结果审核
            // 考核结束
            stepArr: [
                this._sn('P_YS_HR_HRJXF_0000059271'), this._sn('P_YS_HR_HRJXF_0000059218'), this._sn('P_YS_HR_HRJXF_0000059705'), this._sn('P_YS_PF_GZTSYS_0000056343'), this._sn('P_YS_HR_HRJXF_0000059357')
            ],
            halfstepArr: [
                this._sn('P_YS_HR_HRJXF_0001007469'), this._sn('P_YS_HR_HRJXF_0000059218'), this._sn('P_YS_HR_HRJXF_0000130274')
            ],
            activeStep: 2,
            staffid: '',
            talentVisible: false,
            startDate: '',
            endDate: '',
            scoreway: 1, // 评分方式 1加权求和2算数求和3算术平均
            schememode: 0, // 值为4时为价值观,20时为中期回顾方案
            selfRating: false, // 自评/他评
            hasStrips: false,  // 有触犯时，上面组件全部禁用
            radioStrips: false,
            preturnsVisible: false,
            isallowmtr: false,  // 是否显示中期回顾按钮
            schemeid: '',
            isadjust: false, // 是否显示调整分
            adjust: undefined, // 调整分值
            exerciseStatus: '',
            adjustscorename: '',
            tempFormData: '', // 临时数据，与刚进入页面时评分数据一致，用来判断页面评分数据是否修改
            tempComData: '', // 临时数据，与刚进入页面时数据一致，用来判断综合评价是否修改
            activeNames: ''
        }
    },
    computed: {
        indicatorArr () {
            if (this.radarData[0].data.radarInfo.length !== 0) {
                let radarArr = this.radarData[0].data.radarInfo.map(function (item, index) {
                    return {
                        text: `${item.characterName} (${item.percentage}%)`,
                        max: 120,
                        color: '#DBDBDB'
                    }
                })
                let length = 6 - radarArr.length
                if (length < 6) {
                    for (let j = 0; j < length; j++) {
                        radarArr.push({ text: '', max: 120, color: '#DBDBDB' })
                    }
                }
                return radarArr
            }
        },
        overAllTip () {
            if (this.hasStrips) {
                // 请在综合评价内填写举证时间、地点、事件和该事件造成的影响
                return this._sn('P_YS_HR_HRJXF_0000130280')
            } else {
                // 结合自己这一年价值观的践行情况，做个综合评价吧～ 示例：这一年自己面对客户需求第一时间响应，每个客户做项目复盘；坚持每月读一本书，持续优化现有方案；面对工作，不甩锅、不说“差不多就行了”，以实际言行践行三大价值观。
                // 请结合该员工这一年价值观的践行情况，给他/她做个综合反馈吧~
                return this.schememode === 4 ? (this.selfRating ? this._sn('P_YS_HR_HRJXF_0000130283') : this._sn('P_YS_HR_HRJXF_0000130216')) : ''
            }
        },
        // 计算雷达图上面的值的数组
        radarValArr () {
            let radarData = this.radarData
            let outArr = [

            ]
            let intArr
            for (let i = 0; i < radarData.length; i++) {
                if (radarData[i].data.radarInfo.length !== 0) {
                    intArr = [

                    ]
                    outArr.push({ value: intArr })
                    let radarInfo = radarData[i].data.radarInfo
                    if (radarInfo.length < 6) {
                        for (let k = 0; k < 6; k++) {
                            if (k < radarInfo.length) {
                                intArr.push(radarInfo[k].percentage + 20)
                            } else {
                                intArr.push(20)
                            }
                        }
                    } else {
                        for (let j = 0; j < radarInfo.length; j++) {
                            intArr.push(radarInfo[j].percentage + 20)
                        }
                    }
                }
                return outArr
            }
            return [
                { value: [] }
            ]
        },
        option () {
            return {
                calculable: true,
                title: {
                    // text: this._sn('P_YS_HR_HRJXF_0000059395'), // 荣耀雷达图
                    left: '16px',
                    textStyle: {
                        fontWeight: 'normal',
                        fontFamily: '-apple-system,BlinkMacSystemFont,Segoe UI,Roboto,Helvetica Neue,Helvetica,PingFang SC,Hiragino Sans GB,Microsoft YaHei,SimSun,sans-serif'
                    }
                },
                polar: [
                    {
                        indicator: this.radarData[0].data.radarInfo.length === 0 ? this.indicator : this.indicatorArr,
                        nameGap: 9,
                        splitNumber: 6, // 分割段数
                        splitLine: {
                            // 分隔线
                            lineStyle: {
                                color: [
                                    'rgba(223, 223, 223, 0.5)',
                                    'rgba(223, 224, 223, 1)',
                                    'rgba(223, 223, 223, 0.5)',
                                    'rgba(223, 223, 223, 1)',
                                    'rgba(223, 223, 223, 0.5)',
                                    'rgba(223, 223, 223, 1)'
                                ].reverse()
                            }
                        },
                        splitArea: {
                            // 分割区域
                            show: false
                        },
                        axisLine: {
                            // 坐标轴轴线
                            lineStyle: {
                                color: 'rgba(223, 223, 223, 1)'
                            }
                        },
                        center: ['50%', '50%'],
                        radius: '100px'
                    }
                ],
                grid: [

                ],
                series: [
                    {
                        type: 'radar',
                        symbol: 'none',
                        itemStyle: {
                            normal: {
                                color: '#F87571', // 图表中各个图区域的边框线拐点颜色
                                lineStyle: {
                                    color: '#F87571', // 图表中各个图区域的边框线颜色
                                    opacity: 0.5
                                },
                                areaStyle: { type: 'default' }
                            }
                        },
                        areaStyle: {
                            normal: {
                                color: ['#F87571'], // 分隔区域颜色。分隔区域会按数组中颜色的顺序依次循环设置颜色。默认是一个深浅的间隔色。
                                shadowColor: '#F87571',          // 阴影颜色
                                shadowOffsetX: 0,            // 阴影水平方向上的偏移距离。
                                shadowOffsetY: 0,            // 阴影垂直方向上的偏移距离
                                shadowBlur: 10,              // 图形阴影的模糊大小。
                                opacity: 0.5
                            }
                        },
                        data: this.radarValArr
                    }
                ],
                fromList: true
            }
        }
    },
    created: function () {
        this.loading = true
        this.ind = this.$route.query.index
        this.len = this.$route.query.len
        this.staffList = JSON.parse(getStorage('staffList', 'session')) || []
        if (!this.$route.query.sheetid) {
            this.fromList = true
            this.getListData()
        } else {
            this.fromList = false
            this.turn = this.$route.query.turn
            this.sheetid = this.$route.query.sheetid
            Promise.all([this.getData(), this.getBaseInfo()]).then(() => {
                this.loading = false
                this.getOverview().then(res => {
                    // 如果是员工自评， 且是价值观 显示十大天条弹窗,或者保存过但没触犯
                    if (this.selfRating && this.schememode === 4 && this.editable && !this.hasStrips && this.exerciseStatus === '') {
                        this.showStrips()
                    }
                    // 暂存临时数据
                    this.setTempData()
                })
                this.getLastScore()
            }
            ).catch(() => {
                this.loading = false
            })
        }
    },
    mounted: function () {
    },
    methods: {
        runFun (obj) {
            this.$forceUpdate()
        },
        /**
         * @param{String} id key值
         * @param{Array} array 
         */
        getStatusByScore (id, array, numvalue = '') {
            if (id && array && array.length !== 0) {
                // 根据id找到值
                numvalue = array.filter((item) => {
                    return item.id === id
                })[0].standardscore
            }
            if (numvalue !== '') {
                return numvalue >= 3.5 || numvalue <= 1
            } else {
                return false
            }
        },
        selectChange ($event, val, index, ind, idx) {
            let {
                numvalue, id, text
            } = val
            this.listData[index].child[ind].objChild[idx].oappraiseData['008'].numvalue = numvalue
            this.listData[index].child[ind].objChild[idx].oappraiseData['008'].gradeitemid = id
            this.listData[index].child[ind].objChild[idx].oappraiseData['008'].text = text
        },
        // 中期回顾
        toMidReview () {
            // 利用正则去掉ts字段，因为每次查数据时间戳都会变
            let tempFormData = JSON.stringify(this.getParam()).replace(/\:\d{13}\,/g, ':0,')
            let tempComData = JSON.stringify(this.getComData()).replace(/\:\d{13}\,/g, ':0,')
            // 浏览态直接跳中期回顾
            if (!this.editable) {
                this.$router.push({
                    name: 'midReview',
                    query: {
                        schemeid: this.schemeid,
                        staffid: this.staffid
                    }
                })
            } else if (this.editable && tempFormData === this.tempFormData && tempComData === this.tempComData) {
                // 编辑态且数据没有修改过，则直接进入中期回顾页面
                this.$router.push({
                    name: 'midReview',
                    query: {
                        schemeid: this.schemeid,
                        staffid: this.staffid
                    }
                })
            } else { // 若修改过，则提示
                this.$confirm(this._sn('P_YS_HR_HRJXF_0001023704'), {
                    confirmButtonText: this._sn('P_YS_FED_EXAMP_0000019939'),
                    cancelButtonText: this._sn('P_YS_FED_EXAMP_0000020385'),
                    type: 'warning'
                }).then(() => {
                    this.$router.push({
                        name: 'midReview',
                        query: {
                            schemeid: this.schemeid,
                            staffid: this.staffid
                        }
                    })
                }).catch(() => { })
            }
        },
        showIframe (url) {
            this.showIframeDialog = true
            setTimeout(() => {
                this.$refs.iframe.src = url
            }, 5)
        },
        translate,
        // checkDec (value, index, i) {
        //     this.listData[index].child[i].oappraiseData['008'].numvalue = parseFloat(parseFloat(value).toFixed(this.scoreLimit.decimaldigits))
        // },
        recover () {
            let param = {
                sheetid: this.sheetid,
                appriser: this.staffid,
                turn: this.turn
            }
            fetchData(`${window.hrPath}${performance.pbc}/pm/pa/callbackAppraise`, { data: param, type: 'post' }).then(res => {
                if (res.statusCode === 200) {
                    this.$message({
                        type: 'success',
                        message: res.message
                    })
                    this.getData()
                } else {
                    this.$message({
                        type: 'error',
                        duration: 5000,
                        message: res.message
                    })
                }
            }).catch(err => {
                this.$message({
                    type: 'error',
                    duration: 5000,
                    message: err
                })
                throw new Error(err)
            })
        },
        toIndex () {
            if (!this.$route.query.sheetid) {
                this.$router.push({
                    name: 'evaluate'
                })
            } else {
                this.$router.go(-1)
            }
        },
        getlastTurn () {
            this.listData.forEach(item => {
                item.child.forEach((value) => {
                    value.objChild.forEach(i => {
                        // 复制举证
                        this.$set(i.oappraiseData['009'], 'text', this.lastList[i.indicatorid]['009'] || i.oappraiseData['009'].text)
                        if (this.scoreLimit.ruletype === 1) {
                            this.$set(i.oappraiseData['008'], 'gradeitemid', this.lastList[i.indicatorid].gradeid)
                            // i.oappraiseData['008'].gradeitemid = this.lastList[i.indicatorid].gradeid
                        } else {
                            this.$set(i.oappraiseData['008'], 'numvalue', this.lastList[i.indicatorid].numvalue)
                            // i.oappraiseData['008'].numvalue = this.lastList[i.indicatorid].numvalue
                        }
                    })
                })
            })
        },
        changeFun () {
            this.listData = [...this.listData]
        },
        getLastScore () {
            let param = {
                'sheetid': this.sheetid,
                'turn': this.turn
            }
            fetchData(`${window.hrPath}${performance.pbc}/pm/pa/queryLastEvaTurnScoreAll`, { data: param }).then(res => {
                this.lastList = res.data
                // this.getBaseInfo()
                // this.getData()
                // this.getOverview()
            }).catch(err => {
                this.loading = false
                this.$message({
                    type: 'error',
                    duration: 5000,
                    message: err
                })
                throw new Error(err)
            })
        },
        getListData () {
            let param = {
                'schemeid': this.$route.query.schemeid,
                'markFlag': this.$route.query.markFlag
            }
            fetchData(`${window.hrPath}${performance.pbc}/pm/pa/queryStaffByScheme`, { data: param }).then(res => {
                if (res.data.length === 0) {
                    return
                }
                if (this.staffList.length === 0) {
                    res.data.forEach(item => {
                        let obj = { 'sheetid': item.sheetid, 'turn': item.turn, 'editable': item.statusname !== this._sn('P_YS_HR_HRPUBF_0000053739') } // 已考核
                        this.staffList.push(obj)
                    })
                }
                this.sheetid = this.staffList[this.ind].sheetid
                this.turn = this.staffList[this.ind].turn
                let _this = this
                this.$nextTick(() => {
                    Promise.all([_this.getData(), _this.getBaseInfo()]).then(() => {
                        this.loading = false
                        this.getOverview().then(res => {
                            // 如果是员工自评， 且是价值观 显示十大天条弹窗,或者保存过但没触犯
                            if (this.selfRating && this.schememode === 4 && this.editable && !this.hasStrips && this.exerciseStatus === '') {
                                this.showStrips()
                            }
                            // 暂存临时数据
                            this.setTempData()
                        })
                        this.getLastScore()
                    }
                    )
                })
            }).catch(err => {
                this.loading = false
                this.$message({
                    type: 'error',
                    duration: 5000,
                    message: err
                })
                throw new Error(err)
            })
        },
        changeP (flag, index) {
            // if (flag) {
            // this.$emit('change', flag, index)
            // } else {
            //   this.$emit('pre-target', index)
            // }
            if (flag === -1 && parseInt(this.ind) !== 0 || flag === 1 && parseInt(this.ind) !== parseInt(this.len) - 1) {
                this.loading = true
                // this.editable = this.staffList[parseInt(this.ind) + flag].editable
                this.sheetid = this.staffList[parseInt(this.ind) + flag].sheetid
                this.turn = this.staffList[parseInt(this.ind) + flag].turn
                this.editable = this.staffList[parseInt(this.ind) + flag].editable
                this.ind = parseInt(this.ind) + flag
                this.$router.push({
                    query: merge(this.$route.query, { 'index': this.ind })
                })
                // this.lastList = {}
                // this.loading = true
                // this.getListData()
                let _this = this
                //   let p1 = new Promise(function (resolve, reject) {
                //     _this.getBaseInfo()
                //     resolve()
                //   })
                //   let p2 = new Promise(function (resolve, reject) {
                //     _this.getData()
                //     resolve(p1)
                //   })
                //   p2.then(() => {
                //     this.loading = false
                //   }
                // )
                // this.getBaseInfo().then(this.loading = false).fail(this.loading = false)
                // this.getData()
                Promise.all([_this.getData(), _this.getBaseInfo()]).then(() => {
                    this.getLastScore()
                    this.loading = false
                }
                )
                this.getOverview()
                // this.$nextTick(() => {
                // setTimeout(() => {
                //   this.loading = false
                // }, 600)
                // })
                // if (this.changeFlag.getBaseInfo && this.changeFlag.getData && this.changeFlag.getOverview) {
                //   this.loading = false
                // }
            } else {

            }
        },
        setTempData () {
            this.tempFormData = JSON.stringify(this.getParam()).replace(/\:\d{13}\,/g, ':0,')
            this.tempComData = JSON.stringify(this.getComData()).replace(/\:\d{13}\,/g, ':0,')
        },
        getParam () {
            let param = []
            let lists = []
            let flag = 0
            this.flagSave = 0
            // input-number输入视角校验了小数位保存暂时去除
            this.listData.forEach(item => {
                // lists.push(item)
                item.child.forEach((value) => {
                    value.objChild.forEach(i => {
                        if (i.oappraiseData['008'].numvalue !== undefined) {
                            i.oappraiseData['008'].numvalue = parseFloat(parseFloat(i.oappraiseData['008'].numvalue).toFixed(this.scoreLimit.decimaldigits)) ? parseFloat(parseFloat(i.oappraiseData['008'].numvalue).toFixed(this.scoreLimit.decimaldigits)) : ''
                        } else {
                            delete i.oappraiseData['008'].numvalue
                        }
                        lists.push(i)
                    })
                })
            })
            lists.forEach(item => {
                let data = {
                    oappraiseData: item.oappraiseData || {},
                    osheetData: item.osheetData
                }
                // osheetData.push(item.osheetData)
                param.push(data)
                if (item.oappraiseData['008'].numvalue > this.scoreLimit.upperlimit || item.oappraiseData['008'].numvalue < this.scoreLimit.lowerlimit) {
                    flag++
                }
            })
            return {
                param, // 表单数据
                flag
            }
        },
        getComData () {
            let comData = {
                id: this.overviewData[0].id,
                comment: this.comment
            }
            if (this.hasStrips) {
                comData.exercisestatus = this.hasStrips ? 0 : 1
            }
            if (this.isadjust && this.adjust) {
                comData.adjustscore = '' + this.adjust
            }
            return comData
        },
        /**
         * @param{String}
         */
        save (submit) {
            let data = this.getParam()
            let param = data.param
            let flag = data.flag
            this.flagSave = 0
            // TODO 得分应在
            // 之间
            if (flag > 0) {
                this.$message({
                    type: 'error',
                    duration: 5000,
                    message: translate('P_YS_HR_HRJXF_0000059401', [this.scoreLimit.lowerlimit, this.scoreLimit.upperlimit])
                })
            } else {
                this.loading = true
                // 保存指标评价 和 综合评价
                let url = `${performance.pbc}/pm/pa/saveAppraiseData`
                let comUrl = `${performance.pbc}/pm/pa/saveAppraiseComment`
                let comData = this.getComData()
                this.query(comUrl, comData).then(() => {
                    this.query(url, param).then(() => {
                        if (submit === 'submit') {
                            this.$nextTick(() => {
                                this.submit()
                            })
                        } else {
                            this.$message({
                                type: 'success',
                                message: this._sn('P_YS_FED_FW_0000020961') // 保存成功
                            })
                            this.$nextTick(() => {
                                this.getData('needClose', () => {
                                    this.loading = false
                                })
                            })
                        }
                    }).catch(e => {
                        this.loading = false
                        this.$message({
                            type: 'error',
                            duration: 5000,
                            message: e.message
                        })
                    })
                }).catch(e => {
                    this.loading = false
                })
            }
        },
        query (url, param, submit) {
            // let _this = this
            let p = new Promise((resolve, reject) => {
                fetchData(window.hrPath + url, { headers: {}, data: param, type: 'post' }).then(res => {
                    if (res.statusCode === 200) {
                        if (url === `${performance.pbc}/pm/pa/submitAppraise`) {
                            // 提交
                            // this.loading = false
                            this.$message({
                                type: 'success',
                                message: res.message
                            })
                            // this.$router.back()
                            Promise.all([this.getData(), this.getBaseInfo()]).then(() => {
                                this.setTempData()
                                this.loading = false
                            }).catch(() => {
                                this.loading = false
                            })
                            this.getOverview()
                        } else {
                            // this.loading = false
                            this.setTempData()
                            resolve(res)
                        }
                    } else {
                        this.loading = false
                        // save时后端需要拿新的ts时间戳 所以提交成功失败都需要重新请求
                        this.$message({
                            type: 'error',
                            duration: 5000,
                            message: res.message
                        })
                        this.getData('needClose', () => {
                            this.loading = false
                        })
                        reject(res)
                    }
                    // resolve()
                }).catch(err => {
                    this.$message(err.message)
                    this.loading = false
                    reject(err)
                })
            })
            return p
        },
        submit () {
            this.loading = true
            let param = {
                sheetid: this.sheetid,
                appriser: this.staffid
            }
            let msg = []
            this.msg = []
            this.listData.forEach(i => {
                i.child.forEach((value) => {
                    value.objChild.forEach((item, index) => {
                        console.log(item)
                        if (this.scoreLimit.ruletype === 0 || item.scoreRule && item.scoreRule.ruletype === 0) {
                            // && this.msg.indexOf(index) < 0
                            if (!item.oappraiseData['008'].numvalue && item.oappraiseData['008'].numvalue !== 0) {
                                msg.push(item.indiParentName)
                                this.msg.push(index)
                                // item.check = true
                                this.$set(item, 'check', true)
                            }
                        } else {
                            // && this.msg.indexOf(index) < 0
                            if (!item.oappraiseData['008'].gradeitemid) {
                                msg.push(item.indiParentName)
                                this.msg.push(index)
                                this.$set(item, 'check', true)
                            }
                        }
                    })
                })
            })
            if (msg.length > 0 && !this.radioStrips) {
                // this.loading = false
                this.$message({
                    type: 'error',
                    duration: 5000,
                    showClose: true,
                    message: `${this._sn('P_YS_HR_HRJXF_0000059510')}${this.msg.length}${this._sn('P_YS_HR_HRJXF_0000059643')}`
                })
                this.getData('needClose', () => { this.loading = false })
            } else {
                //   this.loading = false
                let url = `${performance.pbc}/pm/pa/submitAppraise`
                this.query(url, param)
            }
        },
        // 请求综合评价数据
        getOverview () {
            let promise = new Promise((resolve, reject) => {
                let param = {
                    sheetid: this.sheetid,
                    turn: this.turn
                }
                fetchData(window.hrPath + performance.pbc + '/pm/pa/queryAppraiseComment', { headers: {}, data: param, type: 'get' }).then(res => {
                    this.changeFlag.getOverview = true
                    if (res.statusCode === 200) {
                        this.overviewData = res.data
                        this.comment = res.data[0].comment ? res.data[0].comment : ''
                        this.hasStrips = res.data[0].exerciseStatus === 0 || false
                        this.exerciseStatus = res.data[0].exerciseStatus === undefined ? '' : res.data[0].exerciseStatus
                        this.radioStrips = this.hasStrips
                        resolve(resolve)
                    }
                }).catch((e) => {
                    this.changeFlag.getOverview = true
                    reject(e)
                })
            })
            return promise
        },
        // 请求雷达图数据
        getAndarData () {
            this.chartLoading = true
            let param = {
                sheetid: this.sheetid,
                periodid: this.periodid
                // sheetid: '52e578804bb24ceabb3a9c4a67791584', // this.sheetid,
                // periodid: 'iqopszg1pm0000000109' // this.periodid
            }
            fetchData(window.hrPath + performance.pbc + '/pm/pa/queryEvaluationBasis', { headers: {}, data: param, type: 'get' }).then(res => {
                if (res.statusCode === 200) {
                    this.chartLoading = false
                    // let rres = [{'msg': '请使用正确授权accessKey', 'code': 'sys.validate.error', 'flag': 1}, {}]
                    if (res.data[0].data) {
                        this.radarData = res.data
                    }
                    if (res.data[1].staffId) {
                        this.chart.setOption(this.option)
                    } else {
                        this.chart2.setOption(this.option)
                    }
                    // } else {
                    //   if (res.data[1].staffId) {
                    //     this.chart.setOption(this.option)
                    //   } else {
                    //     this.chart2.setOption(this.option)
                    //   }
                    // }
                    this.attendData = res.data[1]
                    // this.chart.resize()
                } else {
                    this.chart2.setOption(this.option)
                    this.chartLoading = false
                }
            }).catch(() => {
                this.chart2.setOption(this.option)
                this.chartLoading = false
            })
        },
        // 格式化头像
        getPhoto (objData) {
            // objData.photo = 'http://img3.duitang.com/uploads/item/201511/04/20151104120338_JBXYr.jpeg'
            objData.nickname = ''
            objData.nameBack = ''
            let username = objData.staffname
            let pic = objData.photo || ''
            if (pic.length < 1 && username) {
                // 如果没有个人头像，将默认头像替换为名字后两位并添加背景色截取名字后两位
                let newname = encodeURI(username).replace(/%/g, '')
                var colors = ['#f99a2b', '#eead10', '#06aae1', '#89a8e0', '#0abfb5', '#00ced1', '#f99a2b', '#96bc53', '#00ced1', '#89a8e0']
                if (newname.length > 6) {
                    let lastname, color, colorindex
                    lastname = newname.substr(lastname, 6)
                    color = parseInt(lastname, 16)
                    if (isNaN(color)) {
                        var val = ''
                        for (var i = 0; i < lastname.length; i++) {
                            if (val === '') {
                                val = lastname.charCodeAt(i).toString(16)
                            } else {
                                val += lastname.charCodeAt(i).toString(16)
                            }
                        }
                        color = parseInt(val, 16)
                    }
                    colorindex = color % 10
                    objData.nameBack = colors[colorindex]
                } else {
                    objData.nameBack = colors[10]
                }
                objData.photo = ''
                let name = username.substr(username.length - 2, 2)
                objData.nickname = name
            }
            this.baseList = objData
        },
        // 获取人员基本信息
        getBaseInfo () {
            let data = {
                // 'pk_pe_doc': this.pk_pe_doc,
                'sheetid': this.sheetid
            }
            var p1 = new Promise((resolve, reject) => {
                fetchData(window.hrPath + performance.core + '/pm/system/getPsnSheetInfo', { headers: {}, data, type: 'POST' }).then(data => {
                    // this.loading = false
                    if (data.statusCode === 200) {
                        // this.baseList = data.data
                        this.getPhoto(data.data)
                        this.active = Number(data.data.schemestate)
                        this.staffid = data.data.staffid
                        // return new Promise((resolve, reject) => {

                        // })
                    }
                    resolve()
                }).catch(() => {
                    // this.loading = false
                    resolve()
                })
            })
            return p1
        },
        // 查询指标列表
        getData (type, callback = () => { }) {
            let data = {
                sheetid: this.sheetid,
                turn: this.turn
            }
            var p2 = new Promise((resolve, reject) => {
                fetchData(window.hrPath + performance.pbc + '/pm/pa/queryAppraiseData', {
                    headers: {},
                    data,
                    type: 'get'
                }).then(data => {
                    // _this.loading = false
                    if (data.statusCode === 200) {
                        this.weigData = data.data
                        this.scoreLimit = data.data.scoreLimit
                        this.score = data.data.text || data.data.score
                        this.lastScore = data.data.lastScore
                        this.periodid = data.data.periodid
                        this.editable = data.data.markStatus !== 3
                        this.startDate = data.data.period.fdate
                        this.endDate = data.data.period.tdate
                        this.scoreway = data.data.scoreway
                        this.schememode = data.data.schememode
                        this.schemeid = data.data.sheet.schemeid
                        this.selfRating = data.data['self-rating']
                        this.isallowmtr = data.data.isallowmtr
                        this.isadjust = data.data.isadjust
                        // 设为undefined是置空，若为''则会被转成0
                        this.adjust = data.data.adjustscore !== null ? data.data.adjustscore : undefined
                        this.adjustscorename = data.data.adjustscorename
                        this.handleFun(data.data.sheet.osheetIndis)
                        
                        this.changeFlag.getData = true
                        // this.listData.forEach((item, index) => {
                        //     this.arrBox.push(item.child.length)
                        // })
                        // 获取雷达图数据
                        // this.getAndarData()
                        setTimeout(() => {
                            var standards = document.getElementsByClassName('standard')
                            for (let index = 0; index < standards.length; index++) {
                                ((i) => {
                                    standards[i].onclick = (el) => {
                                        this.showIframe(el.srcElement.parentElement.getAttribute('data-url'))
                                    }
                                })(index)
                            }
                        }, 500)
                        if (type === 'needClose') {
                            callback()
                        }
                    } else {
                        callback()
                        this.$message({
                            type: 'warning',
                            duration: 5000,
                            message: data.message
                        })
                        this.listData = []
                    }
                    resolve()
                }).catch(() => {
                    // this.changeFlag.getData = true
                    callback()
                    resolve()
                    // this.loading = false
                })
            })
            return p2
        },
        // 处理层级关系
        handleFun (obj) {
            let arr = []
            let i = ''
            let num = 0
            obj.forEach((item, index) => {
                // 修改数据，层级关系
                item.evalstandard = item.evalstandard.replace(/\<a/g, '<a class="standard"').replace(/\_blank/g, 'iframe').replace(/http\:/g, 'https\:').replace(/href/g, 'data-url')
                item.detailShow = false
                if (item.actWeight !== undefined) {
                    item.act_weight = item.actWeight * 100
                } else {
                    item.act_weight = ''
                }
                item.osheetData['006'].text = item.osheetData['006'].text || ''
                // 反显
                if (item.scoreRule) {
                    item.scoreRule.ogradeItem && item.scoreRule.ogradeItem.forEach((rule) => {
                        rule.text = rule.gradeitemname
                        rule.numvalue = rule.standardscore
                        delete rule.gradeitemname
                        delete rule.standardscore
                    })
                }
                // 分类权重层级
                if (i !== item.indiParent) {
                    num += 1
                    arr.push({
                        indi_parent: item.indiParent,
                        parentname: item.indiParentName,
                        numtotal: '',
                        indiclass_weight: (item.indiclassWeight * 100).toFixed(0),
                        customId: num,
                        child: []
                    })
                    arr[arr.length - 1].child.push(item)
                    arr[arr.length - 1].numtotal = item.act_weight
                    i = item.indiParent
                } else {
                    arr[arr.length - 1].child.push(item)
                    arr[arr.length - 1].numtotal += item.act_weight
                }
            })
            this.handleObj(arr)
            this.listData = JSON.parse(JSON.stringify(arr.slice(0)))
            this.tempListData = JSON.parse(JSON.stringify(this.listData))
        },
        // 关联目标层级
        /**
         * @des [深层处理]关联目标层级
         * @param {Array} obj 初步处理后的数据
         */
        handleObj (obj) {
            // var ynum = 0
            obj.forEach((item) => {
                let yarr = []
                let y = null
                item.child.forEach((val) => {
                    if (!val.objectiveid) {
                        val.objectiveid = ''
                        val.objectivename = ''
                    }
                    if (y !== val.objectiveid) {
                        // ynum += 1
                        yarr.push({
                            objectivename: val.objectivename,
                            objectiveid: val.objectiveid,
                            objChild: []
                        })
                        yarr[yarr.length - 1].objChild.push(val)
                        y = val.objectiveid
                    } else {
                        yarr[yarr.length - 1].objChild.push(val)
                    }
                })
                item.child = []
                item.child = yarr
            })
        },
        // 返回顶部
        backTop () {
            let timer = this.topTimer
            cancelAnimationFrame(timer)
            timer = requestAnimationFrame(function fn () {
                let oTop = document.body.scrollTop || document.documentElement.scrollTop
                if (oTop > 0) {
                    document.body.scrollTop = document.documentElement.scrollTop = oTop - 50
                    timer = requestAnimationFrame(fn)
                } else {
                    cancelAnimationFrame(timer)
                }
            })
        },
        // 弹出层
        showDialog (row, idx) {
            this.dialogVisible = true
            this.hideClass = true
            this.fendis = false
            if (idx && row.sheetindi_id) {
                //  编辑态
                this.fendis = true
                this.idx = row.sheet
                this.isEdit = true
                this.dialogTitle = this._sn('P_YS_HR_HRJXF_0000059505') // 编辑考核项
                this.targetData = {
                    name: { name: row.indiname, pk: row.indicatorid },
                    type: { name: row.parentname, pk: row.indi_parent },
                    percent: parseFloat(row.act_weight) * 1,
                    plan: row.targetvalue,
                    standard: row.evalstandard,
                    id: row.sheetindi_id,
                    // childId: row.indicatorid,
                    parentId: row.indi_parent,
                    parentList: row.parentname
                }
            } else {
                //  新增态
                this.isEdit = false
                this.dialogTitle = this._sn('P_YS_HR_HRPUBF_0000053759') // 新增考核项
                this.targetData = {
                    name: { name: '', pk: '' },
                    type: { name: '', pk: '' },
                    percent: '',
                    plan: '',
                    standard: '',
                    id: '',
                    // childId: '',
                    parentId: '',
                    parentList: ''
                }
            }
        },
        // 展开收起列表
        detailFun () {
            this.detailShow = !this.detailShow
        },
        changeDetail (index, ind, idx, value) {
            // this.detailShow[index] = !this.detailShow[index]
            // let array = this.listData[index].child
            // this.$set(array[i], 'detailShow', !value)
            // this.$set(this.listData[index], 'child',  array)
            this.listData[index].child[ind].objChild[idx].detailShow = !this.listData[index].child[ind].objChild[idx].detailShow
            let comData = JSON.parse(JSON.stringify(this.listData))
            this.listData = comData
        },
        clearNum (obj) {
            obj = parseFloat(obj)
            if (obj > this.scoreLimit.upperlimit || obj < this.scoreLimit.lowerlimit) {
                // TODO 得分应在之间
                this.$message({
                    type: 'error',
                    duration: 5000,
                    message: translate('P_YS_HR_HRJXF_0000059401', [this.scoreLimit.lowerlimit, this.scoreLimit.upperlimit])
                })
            }
        },
        showTalent () {
            this.talentVisible = true
        },
        // 关闭侧边弹框详情
        shutDetail () {
            this.talentVisible = false
        },
        // 上传触发 十大天条举证
        sendOffend () {
            this.loading = true
            let promise = new Promise((resolve, reject) => {
                fetchData(`${window.hrPath}${performance.pbc}/pm/pa/exerciseRight`, {
                    type: 'post',
                    data: {
                        id: this.overviewData[0].id,
                        exercisestatus: this.hasStrips ? 0 : 1,
                        comment: this.comment
                    }
                }).then(res => {
                    if (res.statusCode === 200) {
                        this.loading = false
                        this.$message({
                            type: 'success',
                            message: res.message
                        })
                        resolve(res)
                    } else {
                        this.loading = false
                        this.$message({
                            type: 'error',
                            duration: 5000,
                            message: res.message
                        })
                        reject(res)
                    }
                }).catch(e => {
                    this.loading = false
                    this.$message({
                        type: 'error',
                        duration: 5000,
                        message: e.message
                    })
                    reject(e)
                })
            })
            return promise
        },
        showStrips () {
            if (this.$refs.strips) {
                this.$refs.strips.show()
            }
        },
        closeStrips () {
            if (this.$refs.strips) {
                this.$refs.strips.close()
            }
        },
        /**
         * @param{Boolean} label: 是否触犯十大天条
         */
        offend (label) {
            this.radioStrips = !label
            // let text = '是否确认未遵守《员工商业行为守则》及公司其它规章制度的要求。确认后，您将无权参与2019年度价值观评价！'
            let text = this._sn('P_YS_HR_HRJXF_0000130240')
            if (!this.selfRating) {
                // text = '是否确认未遵守《员工商业行为守则》及公司其它规章制度的要求。确认后，该员工将无权参与2019年度价值观评价！'
                text = this._sn('P_YS_HR_HRJXF_0000130211')
            }
            text = text.replace(/\d+/g, new Date().getFullYear())
            // 是自评，则有触犯就提交
            if (label) {
                this.$confirm(text, {
                    confirmButtonText: this._sn('P_YS_FED_EXAMP_0000019939'),
                    cancelButtonText: this._sn('P_YS_FED_EXAMP_0000020385'),
                    type: 'warning'
                }).then(() => {
                    this.closeStrips()
                    this.radioStrips = true
                    this.hasStrips = true
                    if (this.selfRating) {
                        this.sendOf()
                        // this.sendOffend().then(res => {
                        //     if (res.statusCode === 200) {
                        //         this.$router.go(-1)
                        //     } else {
                        //         this.$message({
                        //             type: 'error',
                        // duration: 5000,
                        //             message: res.message
                        //         })
                        //     }
                        // })
                    }
                }).catch(() => {
                    this.closeStrips()
                    this.radioStrips = false
                    this.hasStrips = false
                })
            } else {
                this.radioStrips = false
                this.hasStrips = false
                this.closeStrips()
            }
        },
        sendOf () {
            this.sendOffend().then(res => {
                if (res.statusCode === 200) {
                    this.$router.go(-1)
                } else {
                    this.$message({
                        type: 'error',
                        duration: 5000,
                        message: res.message
                    })
                }
            })
        },
        /**
         * @des 打开前轮评分弹框
         */
        toPreTurn () {
            this.preturnsVisible = true
        },
        closePreTurn () {
            this.preturnsVisible = false
        },
        handlerPlaceholder (item) {
            const scoreLimit = item.scoreRule || this.scoreLimit
            return this.translate('P_YS_HR_HRJXF_0000059747', [scoreLimit.lowerlimit, scoreLimit.upperlimit])
        },
        handleChange () {
            
        }
    },
    beforeRouteLeave (to, from, next) {
        removeStorage('staffList')
        next()
    }
}
</script>

<style lang="less">
.nameType {
    margin-top: 10px;
    color: #111111;
    font-size: 14px;
    font-weight: 500;
    line-height: 20px;
    margin-left: 12px;
}
.score-part {
    float: left;
    & + div {
        display: inline-block;
        width: calc(~'100% - 120px');
    }
}
.subscribe {
    background-color: #f7f7f7;
    color: #666666;
    width: 100%;
    padding: 12px;
    margin: 12px 0;
    line-height: 1.5;
}
input[type='number']::-webkit-outer-spin-button,
input[type='number']::-webkit-inner-spin-button {
    -webkit-appearance: none;
}

input[type='number'] {
    -moz-appearance: textfield;
}
@keyframes rotating {
    0% {
        -webkit-transform: rotateZ(0deg);
        transform: rotateZ(0deg);
    }
    100% {
        -webkit-transform: rotateZ(360deg);
        transform: rotateZ(360deg);
    }
}
@-webkit-keyframes rotating {
    0% {
        -webkit-transform: rotateZ(0deg);
        transform: rotateZ(0deg);
    }
    100% {
        -webkit-transform: rotateZ(360deg);
        transform: rotateZ(360deg);
    }
}
#evaluate-detail {
    textarea {
        height: 96px;
        font-size: 14px;
    }
    .mix-textarea {
        position: relative;
        .tips {
            line-height: 34px;
        }
    }
    .unit-main {
        word-wrap: break-word;
        -ms-word-wrap: break-word;
        /*line-height: 20px;*/
    }
    .switch {
        margin-top: 9px;
        text-align: center;
        .el-button {
            margin-left: 0;
            margin-right: 4px;
            font-size: 12px;
            padding: 7px 20px;
        }
        i {
            font-size: 12px;
        }
        > div {
            width: 29px;
            height: 24px;
            cursor: pointer;
            background-color: #f2f2f2;
            line-height: 26px;
            text-align: center;
            border-radius: 2px;
            display: inline-block;
            &:first-child {
                margin-right: 2px;
            }
        }
    }
    .pull-icon {
        transition: transform 0.3s;
        display: inline-block;
        &.trans {
            transform: rotate(-180deg);
            -webkit-transform: rotate(-180deg);
        }
    }
    i.iconfont {
        &.pull-icon {
            transition: transform 0.3s;
            &:hover {
                transform: rotate(180deg);
                -webkit-transform: rotate(180deg);
            }
            &.trans {
                transform: rotate(180deg);
                -webkit-transform: rotate(180deg);
            }
        }
        .el-step__title.is-success {
            color: #00cb7c;
        }
    }
    .el-step__title.is-process {
        color: #00cb7c;
    }
    .el-step__head.is-process {
        color: #00cb7c;
        border-color: #00cb7c;
        .iconfont {
            color: #00cb7c;
            font-size: 20px;
        }
    }
    .el-step__head.is-wait {
        .el-step__icon {
            background: #e2e2e2;
            color: #e2e2e2;
            border-color: #e2e2e2;
        }
    }
    .el-step__head.is-success {
        .el-step__icon.is-text {
            border-color: #00cb7c;
            color: #00cb7c;
            border: 7px solid;
            .el-icon-check:before {
                content: '';
            }
        }
    }
    .el-step__line {
        background: #e2e2e2;
    }
    .el-step__icon {
        width: 20px;
        height: 20px;
    }
    .boxes {
        padding: 0 13px;
        // margin-bottom: 20px;
        line-height: 20px;
        .el-row {
            margin-top: 8px;
        }
        &.el-row:first-of-type {
            margin-top: 0;
        }
        .item-title:first-of-type {
            margin-top: 0;
        }
        .i-title {
            letter-spacing: 0;
        }
    }
    #echart-box {
        margin-left: auto;
        margin-right: auto;
    }
    .tips {
        color: #888888;
    }
    .operate-btn {
        color: #007ace;
        cursor: pointer;
        text-align: right;
        i {
            color: #007ace;
        }
    }
    .el-input-number {
        width: 160px;
        .el-input__inner {
            text-align: left;
        }
    }
    .el-select {
        width: 80px;
    }
    .el-collapse {
        border: none;
        .el-collapse-item__wrap {
            border-bottom: none;
        }
        .el-collapse-item__header {
            overflow: hidden;
        }
    }
    .groups {
        overflow: hidden;
        background: #ffffff;
        border: 1px solid rgba(78, 89, 104, 0.19);
        box-shadow: 0 1px 1px 0 rgba(74, 81, 93, 0.1);
        border-radius: 3px;
    }

    .carge {
        width: 64px;
        height: 64px;
        line-height: 64px;
        color: #ececec;
        text-align: center;
        -webkit-box-align: center;
        -ms-flex-align: center;
        align-items: center;
        -webkit-align-items: center;
        border-radius: 50%;
        vertical-align: bottom;
        font-size: 20px;
        color: #fff;
        background-size: 100%;
        background-repeat: no-repeat;
        background-position: 50% 50%;
    }
    .right {
        text-align: right;
        padding-top: 7px;
        .el-button + .el-button {
            margin-left: 8px;
        }
    }
    .see-list {
        display: inline-block;
        padding: 9px 20px;
        .green-font {
            color: #007ace;
            cursor: pointer;
            &:hover {
                color: #038c5b;
            }
        }
    }
    .log-box {
        padding: 20px 0;
    }
    .big-ref {
        width: 100%;
    }
    .widchange .el-autocomplete {
        width: 100%;
    }
    .widchange .modStyle {
        position: absolute;
        top: 4px;
        left: 2px;
        right: 29px;
        width: auto;
        line-height: 31px;
        &.hideput {
            input {
                background: transparent;
            }
        }
        input {
            border: none;
            height: 32px;
        }
    }
    .iconDisten {
        padding-right: 20px;
        cursor: pointer;
    }
    .el-dialog .el-input .el-input__inner,
    .el-dialog .el-textarea .el-textarea__inner {
        &::-webkit-input-placeholder {
            color: red;
        }
        &::-moz-placeholder {
            /* Mozilla Firefox 19+ */
            color: red;
        }
        &:-moz-placeholder {
            /* Mozilla Firefox 4 to 18 */
            color: red;
        }
        &:-ms-input-placeholder {
            /* Internet Explorer 10-11 */
            color: red;
        }
    }
    .progressCon {
        margin-right: calc(~'22px - 10% ');
    }
    .header-wrap {
        z-index: 5;
        overflow: hidden;
        &.header-fixed + * {
            padding-top: 158px;
        }
        .operation {
            width: 50%;
        }
    }
    .rep-card {
        overflow: hidden;
        color: #555555;
        .card-tit {
            font-size: 16px;
            font-weight: bold;
            padding-right: 14px;
        }
        /*}*/
        .card-info {
            line-height: 150%;
            color: #888888;
            font-size: 12px;
        }
    }
    .rep-nav {
        font-size: 12px;
        .score-part {
            > div {
                display: inline-block;
                color: #555555;
                text-align: center;
                overflow: hidden;
                font-size: 22px;
                line-height: 1;
                padding-left: 10px;
                padding-right: 10px;
                position: relative;
                &:first-of-type {
                    padding-left: 0;
                }
                .tips {
                    font-size: 14px;
                    line-height: 24px;
                    color: #757f8c;
                }
                &:first-child:not(:last-child)::after {
                    content: '';
                    position: absolute;
                    width: 1px;
                    background-color: #979797;
                    opacity: 0.38;
                    height: 70%;
                    top: 15%;
                    right: 0;
                }
            }
        }
        width: 100%;
        z-index: 1000;
    }
}
.iframeDialog {
    .el-dialog {
        width: 81%;
        height: 75%;
        overflow: hidden;
        padding-bottom: 9px;
        .el-dialog__headerbtn {
            z-index: 10;
            top: 15px;
        }
        .el-dialog__header {
            margin-bottom: 10px;
        }
        .el-dialog__body {
            height: 90%;
        }
        .el-scrollbar__wrap {
            overflow-x: hidden;
            position: relative;
            left: -1px;
            .el-scrollbar__view {
                height: 101%;
            }
        }
    }
}
</style>
<style lang="less" scoped>
.en_US {
    #evaluate-detail {
        &.editable {
            .boxes {
                .tips {
                    line-height: 17px;
                }
            }
        }
    }
}
</style>
<style lang="less" scoped>
#evaluate-detail.editable {
    .preturn {
        line-height: 20px;
        font-size: 20px;
        position: relative;
        &:after {
            content: '';
            width: 8px;
            height: 8px;
            border: solid #111;
            border-width: 0px 1px 1px 0px;
            transform: translate(-50%, -50%) rotate(-45deg);
            position: absolute;
            right: -10px;
            top: 50%;
        }
    }
    .boxes {
        .tips {
            line-height: 32px;
        }
    }
    .el-select.selectGrade {
        width: 160px;
    }
    .selectGrade {
        & + span {
            display: none;
        }
    }
    .input-label {
        line-height: 32px;
    }
}
.percent {
    color: #333333;
    position: relative;
    &:before {
        content: '';
        background-color: #bfbfbf;
        position: absolute;
        left: -10px;
        top: 0%;
        height: 100%;
        width: 1px;
    }
}
.bar-title {
    font-size: 16px;
    padding-left: 10px;
    font-family: PingFangSC-Semibold;
}
.contraband {
    font-size: 14px;
    font-weight: 400;
    line-height: 17px;
    color: #108ee9;
    cursor: pointer;
    display: inline;
    .el-radio {
        font-weight: 400;
    }
}
.offend {
    margin-top: 30px;
    font-size: 14px;
    .el-row {
        padding: 10px 16px;
    }
}
.complex {
    line-height: 20px;
    font-size: 14px;
}
#evaluate-detail.sn-container {
    .comp {
        .bar-title {
            &::after {
                opacity: 0;
            }
        }
        .rep-bar {
            border-bottom: none !important;
            padding-bottom: 15px;
            padding-top: 0;
        }
    }
    .complex-scan {
        border-bottom: 1px solid #dee5ec;
    }
}
.el-header {
    padding: 20px 20px 0px 30px;
    background-color: #ffffff;
    border-radius: 3px;
}
.el-container {
    .el-main {
        .scoreName {
            font-size: 14px;
            line-height: 20px;
            padding-bottom: 10px;
            border-bottom: 1px solid #d0d0d0;
        }
        .el-collapse-item{
            border-bottom: 4px solid;
            border-color: #F1F1F1;
        }
    }
    .el-header {
        position: relative;
        padding: 20px 30px 0px 30px;
        z-index: 200;
        background-color: #ffffff;
        .rep-title {
            /*margin-bottom: 20px;*/
            /*font-size: 18px;*/
            /*font-weight: bold;*/
            font-size: 17px;
            line-height: 22px;
            color: #111111;
            padding-bottom: 17px;
            font-family: Hiragino Sans GB;
            span:first-child {
                cursor: pointer;
                font-size: 18px;
                color: #555555;
                &::after {
                    content: '';
                    position: absolute;
                    /*margin: 16px 12px;*/
                    margin: 2px 12px;
                    width: 1px;
                    height: 20px;
                    background-color: #d8d8d8;
                }
            }
            span:last-child {
                margin-left: 25px;
            }
        }
        .rep-step {
            margin-bottom: 20px;
        }
        margin-bottom: 6px;
    }
    .el-main {
        overflow: visible;
        padding: 0;
        .rep-type {
            margin-bottom: 10px;
            border-radius: 3px;
            .rep-bar {
                padding-top: 10px;
                position: relative;
                .bar-tip {
                    position: absolute;
                    left: 0;
                    top: 90%;
                    margin-top: -10px;
                    height: 10px;
                    width: 4px;
                    background: #e87672;
                    border-radius: 2px;
                }
                .bar-title {
                    font-size: 16px;
                    padding-left: 0;
                    margin-right: 23px;
                    display: inline-block;
                    color: #333333;
                    span {
                        padding-left: 10px;
                    }
                }
                .bar-per {
                    display: inline-block;
                    font-size: 14px;
                    /*color: #00C780;*/
                }
            }
            .rep-box {
                padding: 8px 12px 0;
                .boxes.rep-item {
                    /*background: #F2F6F9;*/
                    /*border-radius: 3px;*/
                    border: none;
                    margin-bottom: 4px;
                }
                // padding-top: 17px;
                .rep-item {
                    /*margin-bottom:10px;*/
                    /*border: 1px solid #f6f6f6;*/
                    // border:1px dotted #f6f6f6;
                    font-size: 14px;
                    .percent:before {
                        content: '';
                        top: 10%;
                        height: 80%;
                    }
                    .i-title {
                        position: relative;
                        margin-right: 20px;
                        color: #111111;
                        font-size: 16px;
                        line-height: 17px;
                    }
                    .item-main {
                        .item-unit {
                            padding: 0 0 8px;
                            .unit-title {
                                color: #888888;
                            }
                            .unit-main {
                                line-height: 120%;
                            }
                        }
                    }
                }
                &.my-chart {
                    padding-top: 0;
                }
                &.my-attend {
                    /*width: 44%;*/
                    /*float:right;*/
                    padding-top: 0;
                    > div {
                        text-align: center;
                    }
                }
            }
        }
        .rep-dialog {
            .dialog-main {
                margin-top: 15px;
                .el-form-item {
                    margin-bottom: 10px;
                }
            }
        }
        .item-title {
            /*padding-left:20px;*/
            font-size: 14px;
            /*color: #474D54;*/
            /*letter-spacing: 0;*/
            .per-tit {
                font-weight: bold;
                text-overflow: ellipsis;
                white-space: nowrap;
                overflow: hidden;
            }
            .greyfont {
                color: #ccc;
                font-weight: 400;
                text-overflow: ellipsis;
                white-space: nowrap;
                overflow: hidden;
            }
            .item-per {
                font-size: 20px;
                color: #00c780;
                font-weight: 400;
                text-align: center;
            }
        }
    }
}

.rep-type {
    /*padding:5px 30px 30px 30px;*/
    background-color: #ffffff;
}
#chart-detail {
    // max-width: 700px;
    width: 448px;
    // width: 36%;
    height: 100%;
    position: fixed;
    z-index: 1111;
    top: 0;
    right: 0;
    background-color: #ffffff;
    /*border:1px solid #999999;*/
    box-shadow: 0px 6px 8px 0px rgba(74, 81, 93, 0.25),
        -1px 0px 0px 0px rgba(220, 220, 220, 1);
}
.fade-enter-active,
.fade-leave-active {
    transition: all 0.3s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
    /*opacity: 0;*/
    /*margin-left:0;*/
    transform: translateX(50%);
}
.talent-link {
    color: #007ace;
    cursor: pointer;
    font-size: 12px;
    margin-left: 12px;
}
.e-pre {
    word-wrap: break-word;
    white-space: pre-wrap;
    white-space: -moz-pre-wrap;
    border: none;
    background: none;
    font-size: 14px;
    color: #888;
}
.wheel {
    cursor: pointer;
}
.wheelFlow {
    width: 568px;
    height: 100%;
    position: fixed;
    z-index: 1111;
    top: 0;
    right: 0;
    background-color: #ffffff;
    box-shadow: 0px 6px 8px 0px rgba(74, 81, 93, 0.25),
        -1px 0px 0px 0px rgba(220, 220, 220, 1);
}
</style>
